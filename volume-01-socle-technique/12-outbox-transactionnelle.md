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

La méthode `claimBatch` sélectionne uniquement les événements disponibles sous la limite de tentatives, trie de manière déterministe, applique un lot borné, puis utilise une mise à jour optimiste pour empêcher deux workers d’acquérir durablement le même événement. `availableAt` sert également de bail temporaire afin qu’une interruption ne bloque pas définitivement la livraison.

## 4. Orchestration de livraison

`LedgerOutboxDispatcherService` orchestre un lot réclamé sans imposer de broker particulier. Il reçoit un adaptateur conforme à `LedgerOutboxPublisher`, publie chaque événement, appelle `markPublished` après confirmation, appelle `markFailed` en cas d’erreur, applique un backoff exponentiel borné avec jitter et retourne `{ claimed, published, failed }`.

## 5. Worker périodique

Le dépôt plateforme fournit `LedgerOutboxWorker`, un orchestrateur périodique indépendant du transport final.

Le worker :

- déclenche `dispatchBatch` à intervalle configurable ;
- applique une cadence minimale sûre ;
- ne démarre qu’un seul timer ;
- s’arrête proprement ;
- expose `runOnce()` ;
- empêche deux exécutions locales de se chevaucher ;
- libère toujours le verrou d’exécution, même après erreur ;
- transmet les options de lot, bail, tentatives et backoff au dispatcher.

Le worker n’embarque aucun broker ni secret. Il doit être instancié avec un `LedgerOutboxPublisher` concret au niveau infrastructure. Tant qu’aucun transport réel n’est configuré, il ne doit pas être activé en production.

## 6. Observabilité locale du worker

`LedgerOutboxWorker` expose désormais `getSnapshot()` afin de fournir une vue locale sans dépendance à une solution de métriques externe.

Le snapshot contient :

- état `started` et `running` ;
- nombre de cycles terminés ;
- nombre de cycles ignorés parce qu’un cycle précédent est encore actif ;
- nombre de cycles ayant levé une erreur ;
- date du dernier démarrage et de la dernière fin de cycle ;
- durée du dernier cycle ;
- dernier bilan `{ claimed, published, failed }` ;
- dernière erreur technique, tronquée et non sensible.

Cette vue constitue la source locale destinée au futur export Prometheus/OpenTelemetry, à un endpoint interne de santé ou à un superviseur de processus. Elle ne doit pas exposer de payload métier ni de secret.

## 7. Publication réussie et retry

Après confirmation du transport, `markPublished` bascule l’événement vers `PUBLISHED`, renseigne `publishedAt`, efface `lastError` et conserve l’enregistrement pour audit. En cas d’échec, `markFailed` replanifie l’événement et stocke une erreur technique limitée en taille. La limite de tentatives reste appliquée lors du prochain `claimBatch`.

## 8. Idempotence du consommateur

La livraison reste au moins une fois. Les consommateurs doivent être idempotents et utiliser un identifiant stable pour reconnaître un événement déjà appliqué. Aucun flux financier ne doit supposer une garantie réseau exactement une fois.

## 9. Observabilité minimale de production

Les métriques de production devront au minimum couvrir :

- événements `PENDING` et `FAILED` ;
- âge du plus ancien événement disponible ;
- tentatives par type d’événement ;
- latence `createdAt` → `publishedAt` ;
- débit de publication ;
- événements ayant atteint la limite de tentatives ;
- `claimed`, `published`, `failed` par cycle ;
- durée des cycles ;
- cycles ignorés pour chevauchement ;
- cycles ayant échoué avant retour du dispatcher.

Des alertes doivent être déclenchées en cas de backlog trop ancien, volume anormal ou répétition d’échecs.

## 10. Transport externe

Le broker définitif reste volontairement non imposé. L’adaptateur pourra cibler Kafka, RabbitMQ, NATS, un service cloud équivalent ou un transport partenaire. Il devra implémenter `LedgerOutboxPublisher`, confirmer explicitement la publication, utiliser TLS et une identité de workload, appliquer des timeouts bornés, conserver l’identifiant d’événement et ne jamais stocker ses secrets dans Git.

## 11. État actuel

Le dépôt `mansa-platform` contient désormais :

- le modèle Prisma `OutboxEvent` ;
- la création atomique d’événements avec les écritures ledger ;
- `LedgerOutboxService` et sa réclamation par bail optimiste ;
- `markPublished` et `markFailed` ;
- `LedgerOutboxDispatcherService` ;
- backoff exponentiel borné avec jitter ;
- `LedgerOutboxWorker` périodique sans chevauchement ;
- snapshot local d’observabilité du worker ;
- tests Node couvrant cycle, verrou, reprise après erreur et compteurs d’observabilité.

Restent à construire avant production : l’adaptateur vers le broker choisi, le câblage du worker dans un processus d’infrastructure réel, l’export des métriques et alertes, la dead-letter opérationnelle, les tests PostgreSQL réels de concurrence et les scénarios de reprise après interruption brutale.

## 12. Critères d’acceptation

Le lot outbox sera prêt pour recette production lorsque :

- aucun événement validé en SQL ne peut être perdu après redémarrage ;
- plusieurs workers peuvent fonctionner sans double acquisition durable ;
- une interruption libère l’événement après expiration du bail ;
- un worker local ne lance jamais deux cycles simultanés ;
- succès et échec sont persistés correctement ;
- les événements en limite de tentatives sont détectables ;
- les consommateurs sont idempotents ;
- les métriques, alertes et runbooks de reprise sont validés ;
- les tests PostgreSQL et de concurrence sont automatisés.
