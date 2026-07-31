# Références de transaction

## Objectif

Chaque opération financière exposée à un utilisateur, un commerçant, un partenaire ou un agent doit recevoir une référence stable, lisible et indépendante des identifiants techniques internes.

## Format canonique

```text
MNSA-CC-YYYYMMDD-XXXXXXXXXXXX
```

- `MNSA` : préfixe produit fixe.
- `CC` : code pays ISO alpha-2 en majuscules.
- `YYYYMMDD` : date métier UTC de création.
- `XXXXXXXXXXXX` : partie unique de douze caractères alphanumériques en majuscules.

Exemple :

```text
MNSA-ML-20260731-A1B2C3D4E5F6
```

## Règles métier

1. La référence est créée une seule fois lors de l’acceptation initiale de la commande.
2. Un rejeu idempotent retourne exactement la même référence.
3. La référence ne contient aucune donnée personnelle, aucun numéro de téléphone et aucun identifiant bancaire.
4. La partie unique est produite par un générateur cryptographiquement sûr dans la couche applicative.
5. Une contrainte d’unicité doit être appliquée dans la base de données.
6. La référence reste distincte de l’identifiant primaire, de la clé d’idempotence, de la référence partenaire et de l’identifiant de trace.
7. Les reçus, exports, notifications et interfaces de support utilisent cette référence comme identifiant public principal.

## Contrat du package domaine

Le fichier `packages/domain/src/transaction-reference.ts` fournit :

- `createTransactionReference` pour normaliser et valider les composants ;
- `parseTransactionReference` pour extraire le pays, la date métier et la partie unique ;
- `isTransactionReference` pour une validation sans exception ;
- `InvalidTransactionReferenceError` pour signaler les entrées invalides.

Le package domaine ne génère volontairement pas la partie aléatoire : le générateur dépend de l’environnement d’exécution et doit être injecté dans le cas d’usage applicatif afin de rester testable.

## Critères d’acceptation

- Les codes pays autres que deux lettres ASCII sont refusés.
- Les dates impossibles, telles que le 30 février, sont refusées.
- La partie unique doit contenir exactement douze caractères alphanumériques.
- Les entrées en minuscules sont normalisées en majuscules.
- Le parseur restitue exactement les trois composants métier.
- Les tests automatisés couvrent création, analyse, validation et erreurs.

## Suite d’implémentation

- Ajouter la colonne `public_reference` avec contrainte unique dans le modèle persistant des transactions.
- Injecter un générateur sécurisé dans les cas d’usage de paiement et transfert.
- Propager la référence dans les contrats API, événements, reçus et journaux d’audit.
- Prévoir une stratégie de nouvelle tentative en cas de collision exceptionnelle.
