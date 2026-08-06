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

La fonction `validateLedgerEntries` retourne :

- l’état global `valid` ;
- la devise détectée lorsque toutes les écritures sont homogènes ;
- les totaux débit et crédit en unités mineures ;
- une liste d’erreurs structurées avec un code stable ;
- l’index et le compte concernés lorsque l’erreur porte sur une écriture précise.

Codes initiaux :

- `INSUFFICIENT_ENTRIES` ;
- `NON_POSITIVE_AMOUNT` ;
- `MULTIPLE_CURRENCIES` ;
- `UNBALANCED_TOTALS`.

La fonction booléenne `isLedgerBalanced` reste disponible pour les contrôles simples, mais les modules backend doivent privilégier `validateLedgerEntries` afin de journaliser une cause précise sans exposer de donnée sensible.

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

Les tests unitaires et d’intégration doivent couvrir :

- journal équilibré ;
- moins de deux écritures ;
- montant nul ou négatif ;
- plusieurs devises ;
- débits et crédits différents ;
- répétition idempotente identique ;
- collision idempotente avec contenu différent ;
- concurrence sur le même compte ;
- annulation par compensation ;
- reconstruction des soldes ;
- panne entre écriture SQL et publication d’événement ;
- reprise après incident.

## 9. Éléments restant à construire

Le contrat partagé est disponible, mais la production nécessite encore :

- la persistance PostgreSQL ;
- les contraintes uniques et verrous ;
- le module NestJS de comptabilisation ;
- l’outbox transactionnelle ;
- les projections de soldes ;
- les workers de réconciliation ;
- les tests de charge et de reprise ;
- les tableaux de bord et alertes d’intégrité.
