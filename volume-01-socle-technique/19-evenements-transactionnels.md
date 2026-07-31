# Événements transactionnels

## Objectif

Les événements transactionnels décrivent les faits métier importants produits pendant le cycle de vie d’une transaction. Ils constituent un contrat stable entre le domaine, la persistance, l’audit, les notifications et les intégrations asynchrones.

## Événements initiaux

Le socle définit deux événements canoniques :

- `TRANSACTION_CREATED` : une transaction valide a été créée dans l’état `PENDING` ;
- `TRANSACTION_STATE_CHANGED` : une transaction est passée d’un état canonique à un autre.

Ces événements ne remplacent ni l’état courant de la transaction ni les écritures du grand livre. Ils complètent ces données avec un historique append-only exploitable pour l’audit et les traitements asynchrones.

## Enveloppe canonique

Chaque événement contient :

- un identifiant d’événement non vide ;
- un nom d’événement versionnable ;
- la référence publique de transaction normalisée ;
- le type de transaction ;
- la date UTC du fait métier ;
- une charge utile minimale ne contenant aucun secret.

L’événement de création expose l’état initial `PENDING`. L’événement de transition expose les champs `from` et `to`.

## Invariants

- Un événement est immuable après sa création.
- La date et la charge utile sont copiées avant exposition.
- Une transition doit contenir deux états différents.
- Les identifiants et références vides sont refusés.
- Les données sensibles, jetons, numéros de carte complets et secrets partenaires sont interdits dans la charge utile.
- La publication externe intervient uniquement après la validation de la transaction de base de données.

## Persistance et publication

La cible d’architecture est un modèle outbox transactionnel :

1. le cas d’usage modifie l’agrégat ;
2. l’état courant, l’historique et l’événement outbox sont écrits dans la même transaction de base de données ;
3. un worker publie ensuite l’événement vers le bus ;
4. les consommateurs traitent l’événement de manière idempotente ;
5. les échecs sont rejoués sans dupliquer les effets métier.

Cette séquence évite de confirmer une transaction sans avoir enregistré l’événement correspondant, ou de publier un événement avant l’écriture durable de l’état.

## Versionnement

Le nom fonctionnel reste stable. Lorsqu’un changement incompatible de charge utile devient nécessaire, une version explicite doit être ajoutée à l’enveloppe ou au schéma publié. Les consommateurs ne doivent jamais dépendre de champs non documentés.

## Critères d’acceptation

- Les deux événements initiaux peuvent être créés depuis le package domaine.
- Les références sont normalisées en majuscules.
- Les objets produits sont immuables.
- Un identifiant vide est refusé.
- Une transition d’un état vers lui-même est refusée.
- Les tests couvrent la création, la transition, l’immuabilité et les erreurs.

## Cohérence avec le code

L’implémentation de référence se trouve dans :

- `packages/domain/src/transaction-event.ts` ;
- `packages/domain/src/index.ts` ;
- `packages/domain/test/transaction-event.test.mjs`.

## Étapes suivantes

- Relier les événements à l’agrégat sans coupler le domaine au bus.
- Définir le port d’outbox et l’unité de travail.
- Ajouter un événement de règlement comptable après écriture équilibrée.
- Définir les politiques de rétention, rejeu et dead-letter queue.
