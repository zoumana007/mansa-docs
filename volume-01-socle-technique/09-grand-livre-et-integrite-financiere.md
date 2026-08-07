# Grand livre et intégrité financière

## 1. Objectif

Le grand livre est la source de vérité de tous les mouvements financiers de Mansa. Il applique la partie double et interdit toute modification d’une écriture publiée.

## 2. Invariants obligatoires

Une transaction comptable valide respecte simultanément les règles suivantes :

1. elle contient au moins deux écritures ;
2. chaque montant est strictement positif ;
3. toutes les écritures utilisent la même devise ;
4. la somme des débits est exactement égale à la somme des crédits ;
5. une clé d’idempotence unique empêche toute double comptabilisation ;
6. une référence métier et un identifiant de corrélation permettent de relier la comptabilité au parcours applicatif ;
7. une écriture publiée est immuable ;
8. toute correction passe par une transaction compensatoire liée à l’originale.

Les montants sont toujours stockés en unités mineures entières. Aucun nombre flottant n’est autorisé dans les calculs financiers.

## 3. Validation partagée

Le contrat `packages/contracts/src/ledger.ts` expose une validation déterministe commune aux applications et au backend.

### 3.1 Validation des comptes comptables

La fonction `validateLedgerAccount` valide désormais la structure minimale d’un compte avant persistance. Elle contrôle :

- un identifiant de compte non vide ;
- un code comptable de 3 à 64 caractères, limité aux majuscules, chiffres et séparateurs `:`, `_`, `-` ;
- la présence d’un `ownerId` pour tout compte qui n’appartient pas directement à la plateforme ;
- un code pays ISO 3166-1 alpha-2 en majuscules ;
- un nom de compte non vide ;
- un horodatage `createdAt` interprétable comme date-heure.

Les erreurs utilisent les codes stables suivants :

- `INVALID_ACCOUNT_ID` ;
- `INVALID_ACCOUNT_CODE` ;
- `MISSING_OWNER_ID` ;
- `INVALID_COUNTRY_CODE` ;
- `INVALID_ACCOUNT_NAME` ;
- `INVALID_CREATED_AT`.

Un compte `PLATFORM` peut volontairement ne pas avoir de `ownerId`, alors qu’un compte `USER`, `MERCHANT`, `PARTNER` ou `PUBLIC_BODY` doit toujours référencer son propriétaire logique.

### 3.2 Validation des écritures

La fonction `validateLedgerEntries` retourne :

- l’état global `valid` ;
- la devise détectée lorsque toutes les écritures sont homogènes ;
- les totaux débit et crédit en unités mineures ;
- une liste d’erreurs structurées avec un code stable ;
- l’index et le compte concernés lorsque l’erreur porte sur une écriture précise.

Codes initiaux des écritures :

- `INSUFFICIENT_ENTRIES` ;
- `NON_POSITIVE_AMOUNT` ;
- `MULTIPLE_CURRENCIES` ;
- `UNBALANCED_TOTALS`.

La fonction booléenne `isLedgerBalanced` reste disponible pour les contrôles simples, mais les modules backend doivent privilégier `validateLedgerEntries` afin de journaliser une cause précise sans exposer de donnée sensible.

### 3.3 Validation des commandes de publication

La fonction `validatePostLedgerTransactionCommand` valide l’enveloppe métier avant toute persistance. Elle compose la validation financière des écritures avec les contrôles suivants :

- référence métier non vide ;
- type de transaction non vide ;
- clé d’idempotence d’au moins huit caractères ;
- identifiant de corrélation non vide ;
- code pays ISO 3166-1 alpha-2 en majuscules ;
- horodatage `occurredAt` interprétable comme date-heure ;
- écritures respectant tous les invariants de partie double.

Les erreurs de commande utilisent des codes séparés et stables :

- `INVALID_REFERENCE` ;
- `INVALID_TRANSACTION_TYPE` ;
- `INVALID_IDEMPOTENCY_KEY` ;
- `INVALID_CORRELATION_ID` ;
- `INVALID_COUNTRY_CODE` ;
- `INVALID_OCCURRED_AT` ;
- `INVALID_ENTRIES`.

### 3.4 Validation des commandes d’annulation

La fonction `validateReverseLedgerTransactionCommand` valide une demande de compensation avant que le backend ne recherche ou verrouille la transaction d’origine. Elle exige :

- un identifiant de transaction d’origine non vide ;
- un code motif non vide ;
- un motif explicite non vide ;
- une clé d’idempotence d’au moins huit caractères ;
- un identifiant de corrélation non vide.

Les erreurs d’annulation disposent de leur propre catalogue :

- `INVALID_TRANSACTION_ID` ;
- `INVALID_REASON_CODE` ;
- `INVALID_REASON` ;
- `INVALID_IDEMPOTENCY_KEY` ;
- `INVALID_CORRELATION_ID`.

Cette validation reste volontairement pure et sans accès réseau ni base de données. Le service transactionnel doit encore vérifier que la transaction existe, qu’elle est `POSTED`, qu’elle n’a pas déjà été compensée, que l’appelant est autorisé et que la clé d’idempotence n’est pas réutilisée avec un contenu différent.

## 4. Cycle de vie

Une transaction comptable suit les états :

- `PENDING` : reçue et en cours de validation ;
- `POSTED` : validée, séquencée et publiée ;
- `REVERSED` : compensée par une nouvelle transaction ;
- `REJECTED` : refusée avant publication.

Le passage à `POSTED` doit être atomique avec :

- l’écriture de toutes les lignes ;
- l’attribution de la séquence ;
- la création ou mise à jour des projections de solde ;
- la publication de l’événement transactionnel dans une outbox ;
- l’enregistrement de l’audit technique.

Une annulation ne modifie ni ne supprime les écritures d’origine. Elle génère une nouvelle transaction de compensation qui inverse les débits et crédits, conserve la devise et relie explicitement les deux transactions.

## 5. Idempotence et concurrence

La combinaison du domaine métier, de la clé d’idempotence et du propriétaire logique doit être protégée par une contrainte unique en base.

Le traitement doit :

1. ouvrir une transaction SQL ;
2. rechercher une transaction existante avec la même clé ;
3. retourner le résultat existant lorsque la requête est strictement identique ;
4. rejeter la requête lorsque la clé est réutilisée avec un contenu différent ;
5. verrouiller les comptes ou agrégats nécessaires ;
6. valider la partie double ;
7. publier les écritures et l’outbox ;
8. valider la transaction SQL.

Pour une annulation, le verrouillage doit aussi empêcher deux compensations concurrentes de la même transaction d’origine.

## 6. Soldes et projections

Les soldes affichés sont des projections reconstruisibles à partir des écritures. Ils ne remplacent jamais le grand livre.

Au minimum, chaque compte expose :

- solde disponible ;
- solde en attente ;
- devise ;
- date et séquence de calcul.

Toute divergence entre projection et écritures déclenche une alerte, bloque les opérations risquées et ouvre un incident de réconciliation.

## 7. Réconciliation

La réconciliation compare quotidiennement :

- le grand livre Mansa ;
- les fichiers ou API des banques ;
- les opérateurs Mobile Money ;
- les processeurs cartes ;
- les règlements commerçants ;
- les comptes de frais, suspens et réserves.

Chaque écart possède un statut, une cause, un propriétaire, une échéance et une piste d’audit. Une correction manuelle exige une justification et, selon le montant ou le risque, une double validation.

## 8. Tests minimaux

Les tests runtime du package contrats couvrent désormais :

- compte comptable valide ;
- champs invalides d’un compte comptable ;
- compte plateforme sans propriétaire logique ;
- journal équilibré ;
- moins de deux écritures ;
- montant nul ou négatif ;
- plusieurs devises ;
- débits et crédits différents ;
- commande de publication complète ;
- champs obligatoires de commande de publication invalides ;
- propagation d’une erreur financière vers la validation de commande ;
- commande d’annulation complète ;
- champs obligatoires de commande d’annulation invalides.

Les tests backend et d’intégration devront encore couvrir :

- unicité réelle des codes de comptes ;
- existence et statut du propriétaire logique ;
- cohérence entre pays, devise et configuration du produit ;
- répétition idempotente identique ;
- collision idempotente avec contenu différent ;
- concurrence sur le même compte ;
- annulation par compensation ;
- double demande d’annulation concurrente ;
- tentative d’annulation d’une transaction inexistante, rejetée ou déjà compensée ;
- reconstruction des soldes ;
- panne entre écriture SQL et publication d’événement ;
- reprise après incident.

## 9. Éléments restant à construire

Le contrat partagé et ses validations runtime des comptes, publications et annulations sont disponibles, mais la production nécessite encore :

- la persistance PostgreSQL ;
- les contraintes uniques et verrous ;
- le module NestJS de comptabilisation ;
- l’outbox transactionnelle ;
- les projections de soldes ;
- les workers de réconciliation ;
- les tests de charge et de reprise ;
- les tableaux de bord et alertes d’intégrité.
