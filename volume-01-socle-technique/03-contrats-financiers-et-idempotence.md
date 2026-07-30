# Volume 1 — Contrats financiers et idempotence

## 1. Représentation monétaire

Tous les montants sont exprimés en unités mineures entières. Le franc CFA BCEAO (`XOF`) ne possède pas de subdivision utilisée dans la plateforme : `1 XOF` est donc représenté par `1`. Pour les devises à deux décimales, `10,50 EUR` est représenté par `1050`.

Le contrat partagé `Money` contient :

- `amountMinor` : entier signé en précision arbitraire ;
- `currency` : code de devise explicitement supporté.

Les opérations arithmétiques refusent toute combinaison de devises différentes. Les nombres flottants sont interdits dans les soldes, frais, commissions, limites et transactions.

## 2. Cycle de vie d’une transaction

Les types initiaux sont : transfert, paiement, dépôt, retrait, remboursement, frais et extourne.

Les états normalisés sont :

1. `PENDING` : intention acceptée mais non autorisée ;
2. `AUTHORIZED` : contrôles et fonds validés ;
3. `PROCESSING` : traitement interne ou partenaire en cours ;
4. `SUCCEEDED` : opération définitivement réussie ;
5. `FAILED` : échec définitif ;
6. `CANCELLED` : annulation avant exécution définitive ;
7. `REVERSED` : opération réussie puis compensée.

Les états `SUCCEEDED`, `FAILED`, `CANCELLED` et `REVERSED` sont finaux. Une correction financière après succès crée une nouvelle opération compensatoire ; elle ne modifie jamais silencieusement l’écriture d’origine.

## 3. Idempotence

Toute commande pouvant créer un effet financier exige une clé d’idempotence fournie par le client. La clé :

- contient entre 16 et 128 caractères ASCII sûrs ;
- est unique pour un acteur, une opération et une période de conservation ;
- est enregistrée avec l’empreinte canonique de la requête et la réponse obtenue ;
- renvoie la réponse initiale lors d’une répétition identique ;
- produit une erreur de conflit si la même clé est réutilisée avec un contenu différent.

Le serveur conserve les clés au minimum pendant la durée maximale des réessais mobiles, TPE et partenaires. La durée exacte est configurable par environnement et domaine.

## 4. Contrat minimal retourné

Une référence de transaction expose :

- identifiant public non séquentiel ;
- clé d’idempotence ;
- type et état ;
- montant et devise ;
- dates de création et de dernière modification au format ISO 8601 UTC.

Les identifiants internes, informations de compensation et données sensibles des partenaires ne sont jamais exposés directement.

## 5. Cohérence avec le code

La source de vérité technique de ces primitives se trouve dans :

- `packages/contracts/src/money.ts` ;
- `packages/contracts/src/idempotency.ts` ;
- `packages/contracts/src/transaction.ts`.

Tout changement de nom, état ou règle doit mettre à jour simultanément cette documentation, les contrats TypeScript, les schémas OpenAPI et les tests de compatibilité.

## 6. Validations requises

- compilation TypeScript stricte ;
- tests des opérations monétaires et du rejet des devises différentes ;
- tests des bornes et caractères de clé d’idempotence ;
- tests de reconnaissance des états finaux ;
- test de répétition d’une commande sans double écriture ;
- test de conflit pour une clé réutilisée avec une requête différente.
