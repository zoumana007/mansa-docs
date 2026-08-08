# Outbox transactionnelle et livraison fiable des événements

## 1. Objet

Ce document décrit le mécanisme d’outbox utilisé par Mansa pour découpler le commit SQL d’une opération métier de la publication vers un bus ou un broker externe. L’objectif est d’éviter la perte d’événements lorsqu’une transaction financière est confirmée en base mais que le transport événementiel est momentanément indisponible.

Le principe est impératif : la transaction métier et la création de l’événement outbox sont validées dans le même commit SQL. La publication externe intervient uniquement après ce commit.

## 2. Modèle persistant

Le modèle `OutboxEvent` du schéma Prisma conserve notamment :

- `id` : identifiant unique de l’événement ;
- `aggregateType` et `aggregateId` : agrégat métier concerné ;
- `eventType` : type d’événement versionné ;
- `payload` : charge utile sérialisable ;
- `status` : `PENDING`, `PUBLISHED` ou `FAILED` ;
- `attempts` : nombre de tentatives de prise en charge ;
- `availableAt` : instant à partir duquel l’événement peut être réclamé ;
- `publishedAt` : instant de publication réussie ;
- `lastError` : dernière erreur technique, tronquée et non sensible ;
- `transactionId` : lien éventuel avec une transaction du ledger ;
- `createdAt` et `updatedAt`.

Aucun secret, jeton d’accès, PAN complet, donnée KYC ou donnée fournisseur sensible ne doit être copié dans `payload` ou `lastError`.

## 3. Prise en charge concurrente

Le service `LedgerOutboxService` du dépôt plateforme fournit la mécanique de réclamation concurrente.

La méthode `claimBatch` :

1. sélectionne uniquement les événements `PENDING` ou `FAILED` dont `availableAt` est échu ;
2. exclut les événements ayant atteint le nombre maximal de tentatives ;
3. trie de manière déterministe par disponibilité, création puis identifiant ;
4. applique une limite bornée au lot ;
5. tente une mise à jour optimiste sur chaque candidat en vérifiant l’identifiant, le statut, le nombre de tentatives attendu et la disponibilité ;
6. incrémente `attempts` et repousse `availableAt` jusqu’à la fin du bail de traitement ;
7. ne retourne que les événements effectivement réclamés par le worker courant.

Cette stratégie empêche deux workers ayant lu le même candidat de l’acquérir simultanément dans le cas nominal. Le champ `availableAt` sert aussi de bail temporaire : un worker mort ou interrompu n’immobilise pas définitivement l’événement.

## 4. Orchestration de livraison

`LedgerOutboxDispatcherService` orchestre désormais un lot réclamé sans imposer de broker particulier.

Il reçoit un adaptateur conforme au contrat `LedgerOutboxPublisher`, publie chaque événement puis :

- appelle `markPublished` uniquement après confirmation de l’adaptateur ;
- appelle `markFailed` si la publication lève une erreur ;
- calcule un délai de retry exponentiel borné ;
- applique un jitter configurable pour éviter que plusieurs workers se resynchronisent après une panne ;
- retourne un bilan `{ claimed, published, failed }` exploitable par le futur worker et l’observabilité.

Le dispatcher reste volontairement indépendant de Kafka, RabbitMQ, NATS ou d’un service cloud. Le choix de transport sera fourni par un adaptateur dédié.

## 5. Publication réussie

Après confirmation du broker ou du transport cible, le dispatcher appelle `markPublished`.

L’opération :

- ne cible que les événements encore `PENDING` ou `FAILED` ;
- bascule le statut vers `PUBLISHED` ;
- renseigne `publishedAt` ;
- efface `lastError` ;
- conserve l’enregistrement pour l’audit et la réconciliation.

Un événement `PUBLISHED` ne doit pas être remis en file par un simple retry technique.

## 6. Échec, backoff et retry

Lorsqu’une publication échoue, `markFailed` :

- passe l’événement à `FAILED` ;
- calcule une nouvelle date `availableAt` selon le délai de retry fourni ;
- conserve le nombre de tentatives déjà incrémenté lors du claim ;
- enregistre une erreur technique limitée en taille.

Le dispatcher fournit désormais un backoff exponentiel borné avec jitter configurable. La limite de tentatives reste portée par `claimBatch`. Une fois cette limite atteinte, l’événement reste visible pour les opérations de support, d’alerte et de reprise manuelle contrôlée.

## 7. Idempotence du consommateur

L’outbox garantit la conservation et la reprise côté producteur, mais le transport reste fondé sur une livraison au moins une fois. Les consommateurs doivent donc être idempotents.

Chaque événement doit exposer un identifiant stable permettant au consommateur de détecter une livraison déjà appliquée. Les traitements financiers, notifications et intégrations partenaires ne doivent jamais supposer une livraison exactement une fois fournie par le réseau.

## 8. Observabilité minimale

Les métriques de production devront au minimum couvrir :

- nombre d’événements `PENDING` ;
- âge du plus ancien événement disponible ;
- nombre d’événements `FAILED` ;
- nombre de tentatives par type d’événement ;
- latence entre `createdAt` et `publishedAt` ;
- débit de publication ;
- nombre d’événements arrivés au maximum de tentatives ;
- compteurs `claimed`, `published` et `failed` par cycle de worker.

Des alertes doivent être déclenchées si l’âge ou le volume de backlog dépasse les seuils configurés, ou si un type d’événement accumule des échecs répétés.

## 9. Transport externe

Le socle actuel ne choisit volontairement pas encore un broker définitif. Le futur adaptateur pourra cibler Kafka, RabbitMQ, NATS, un service cloud équivalent ou un transport partenaire, sans modifier la règle d’atomicité du ledger.

Le transport devra :

- implémenter `LedgerOutboxPublisher` ;
- confirmer explicitement la publication avant `markPublished` ;
- utiliser TLS et une identité de workload adaptée ;
- ne jamais stocker ses secrets dans Git ;
- respecter les clés d’idempotence et les identifiants d’événement ;
- exposer des timeouts configurables ;
- propager l’identifiant de corrélation lorsque disponible.

Les retries de transport bas niveau doivent être courts et bornés afin de ne pas contourner la politique de retry persistée de l’outbox.

## 10. État actuel

Le dépôt `mansa-platform` contient désormais :

- le modèle Prisma `OutboxEvent` ;
- la création atomique d’événements lors de la publication et de la compensation du ledger ;
- `LedgerOutboxService` pour réclamer un lot par bail optimiste ;
- la gestion de succès via `markPublished` ;
- la gestion d’échec et de replanification via `markFailed` ;
- `LedgerOutboxDispatcherService` pour publier un lot via un adaptateur de transport ;
- un backoff exponentiel borné avec jitter ;
- des tests Node couvrant claim, concurrence optimiste, succès, échec, orchestration et calcul du backoff.

Restent à construire avant production : le processus périodique ou worker dédié, l’adaptateur vers le broker choisi, les métriques et alertes, la dead-letter opérationnelle, les tests PostgreSQL réels de concurrence et les scénarios de reprise après interruption brutale.

## 11. Critères d’acceptation

Le lot outbox sera considéré prêt pour la recette de production lorsque :

- aucun événement créé dans une transaction SQL validée ne peut être perdu après redémarrage ;
- plusieurs workers peuvent fonctionner sans double acquisition durable ;
- une interruption pendant le traitement libère automatiquement l’événement après expiration du bail ;
- un succès marque l’événement `PUBLISHED` une seule fois ;
- un échec planifie un retry contrôlé avec backoff borné ;
- les événements au maximum de tentatives sont détectables et exploitables ;
- les consommateurs sont idempotents ;
- les métriques, alertes et runbooks de reprise sont validés ;
- les tests d’intégration PostgreSQL et de concurrence sont automatisés.
