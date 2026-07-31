# Unité de travail et outbox transactionnelle

## Objectif

Une opération financière ne doit jamais être persistée sans que l’événement destiné aux traitements asynchrones soit lui aussi enregistré. L’unité de travail transactionnelle définit la frontière atomique entre l’écriture de l’agrégat `Transaction` et l’ajout de son événement dans l’outbox.

## Problème traité

Une séquence naïve consistant à sauvegarder la transaction puis à publier directement un événement peut produire deux états incohérents :

- la transaction est enregistrée mais la publication échoue ;
- l’événement est publié alors que la transaction n’a pas été validée.

La cible est donc une écriture unique en base : la transaction et la ligne d’outbox sont validées ou annulées ensemble.

## Contrat du domaine

Le port `TransactionUnitOfWork` expose `saveAndEnqueue(transaction, event, recordedAt)`.

Cette opération doit :

1. persister l’état courant de l’agrégat ;
2. enregistrer l’événement complet dans l’outbox ;
3. conserver la date d’enregistrement ;
4. valider les deux écritures dans la même transaction de stockage ;
5. ne publier aucun message réseau dans cette transaction.

## Modèle d’enregistrement

`TransactionOutboxRecord` contient :

- l’événement métier immuable ;
- `recordedAt`, date à laquelle l’événement a été placé dans l’outbox.

L’adaptateur de production ajoutera les informations techniques nécessaires : identifiant de ligne, statut, nombre de tentatives, date de prochaine tentative, date de publication et dernière erreur nettoyée de toute donnée sensible.

## Publication différée

Un worker indépendant lit les lignes non publiées, envoie les événements au bus puis marque les lignes comme publiées. Il doit supporter :

- la reprise après interruption ;
- les tentatives avec délai progressif ;
- une file d’échec après dépassement du nombre maximal de tentatives ;
- l’idempotence côté consommateur ;
- des métriques de retard, d’échec et de débit.

## Implémentation actuelle

Le package domaine fournit :

- `TransactionUnitOfWork`, contrat indépendant de Prisma et du transport ;
- `TransactionOutboxRecord`, représentation minimale d’un élément d’outbox ;
- `InMemoryTransactionUnitOfWork`, double de test qui conserve les transactions et événements dans l’ordre.

L’implémentation en mémoire sert uniquement aux tests et au développement. Elle ne constitue pas une garantie transactionnelle durable.

## Cohérence avec le code

L’implémentation de référence se trouve dans :

- `packages/domain/src/transaction-outbox.ts` ;
- `packages/domain/src/index.ts` ;
- `packages/domain/test/transaction-outbox.test.mjs`.

## Critères d’acceptation

- le domaine n’importe ni Prisma, ni NestJS, ni client de messagerie ;
- la transaction et l’événement sont remis au même port ;
- les événements gardent leur ordre d’enregistrement ;
- les dates stockées ne partagent pas une référence mutable avec l’appelant ;
- l’adaptateur de production utilise une transaction de base de données réelle ;
- aucune charge utile sensible n’est écrite dans les journaux d’erreur.

## Étapes suivantes

- refactorer `TransactionService` pour utiliser l’unité de travail ;
- définir le schéma Prisma de l’outbox ;
- créer l’adaptateur Prisma et le worker de publication ;
- ajouter les tests de reprise, concurrence et idempotence ;
- documenter les contrats de consommation des événements.
