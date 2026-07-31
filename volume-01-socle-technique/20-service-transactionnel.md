# Service transactionnel du domaine

## Objectif

Le service transactionnel orchestre la création et l’évolution des transactions sans dépendre d’un framework, d’une base de données ou d’un bus de messages précis. Il relie l’agrégat `Transaction`, le port de persistance et le port de publication d’événements.

## Responsabilités

Le service couvre deux cas d’usage initiaux :

- créer une transaction valide, vérifier l’unicité de sa référence, la persister puis publier `TRANSACTION_CREATED` ;
- charger une transaction existante, appliquer une transition autorisée, la persister puis publier `TRANSACTION_STATE_CHANGED`.

Il ne contient aucune logique de transport HTTP, aucune dépendance NestJS et aucun accès Prisma direct.

## Dépendances injectées

Le service reçoit :

- `TransactionRepository` pour lire et sauvegarder l’agrégat ;
- `TransactionEventPublisher` pour publier les événements ;
- une fabrique d’identifiants d’événements ;
- une horloge injectable afin de rendre les tests déterministes.

## Erreurs métier

- `TransactionAlreadyExistsError` lorsqu’une référence est déjà utilisée ;
- `TransactionNotFoundError` lorsqu’une transition vise une transaction absente ;
- les erreurs de montant, de référence et de transition restent portées par les objets métier existants.

## Séquence de création

1. rechercher la référence ;
2. refuser tout doublon ;
3. créer l’agrégat dans l’état `PENDING` ;
4. sauvegarder l’agrégat ;
5. publier l’événement de création ;
6. retourner l’agrégat créé.

## Séquence de transition

1. charger l’agrégat ;
2. refuser une référence inconnue ;
3. mémoriser l’état précédent ;
4. appliquer la transition canonique ;
5. sauvegarder l’agrégat ;
6. publier l’événement de changement d’état ;
7. retourner l’agrégat mis à jour.

## Limite actuelle

L’implémentation actuelle exprime clairement les ports et l’orchestration, mais la garantie atomique entre persistance et publication doit être assurée par l’adaptateur d’infrastructure. La cible est un outbox transactionnel afin que l’écriture métier et l’enregistrement de l’événement soient validés dans une même transaction de base de données.

## Critères d’acceptation

- une création valide est persistée et produit exactement un événement ;
- une référence existante est refusée ;
- une transition valide met à jour l’état et produit un événement ;
- une transaction absente est signalée explicitement ;
- les tests utilisent une horloge, un dépôt et un publisher en mémoire ;
- aucune dépendance d’infrastructure n’est importée par le package domaine.

## Cohérence avec le code

L’implémentation de référence se trouve dans :

- `packages/domain/src/transaction-event-publisher.ts` ;
- `packages/domain/src/transaction-service.ts` ;
- `packages/domain/src/index.ts` ;
- `packages/domain/test/transaction-service.test.mjs`.

## Étapes suivantes

- définir le port d’unité de travail et l’outbox ;
- ajouter l’adaptateur Prisma du dépôt transactionnel ;
- relier le service à un module applicatif de l’API Gateway ;
- ajouter l’idempotence des commandes de création ;
- tracer l’acteur, le canal et la corrélation sans exposer de données sensibles.
