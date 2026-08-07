# Validation des projections de solde

## 1. Objectif

Les soldes affichés par Mansa sont des projections reconstruisibles du grand livre. Ils améliorent les performances de lecture mais ne remplacent jamais les écritures comptables comme source de vérité.

Ce lot ajoute une validation runtime dédiée dans `packages/contracts/src/ledger-balance.ts` afin de détecter les projections structurellement incohérentes avant exposition à une application, une API ou un worker de réconciliation.

## 2. Contrat validé

La validation s’applique à `LedgerBalance` et contrôle quatre invariants minimaux :

1. `accountId` doit être non vide ;
2. `available` et `pending` doivent utiliser exactement la même devise ;
3. le montant `pending` ne peut jamais être négatif ;
4. `asOf` doit être un horodatage interprétable comme date-heure.

Le montant `available` peut être négatif. Cette règle est volontaire : certains produits peuvent autoriser un découvert, un décalage de règlement ou une exposition temporaire. L’autorisation métier d’un solde disponible négatif reste donc du ressort du produit, des limites et de la politique de risque, et non du contrat comptable générique.

## 3. API de validation

Le module expose :

- `validateLedgerBalance(balance)` ;
- `LEDGER_BALANCE_VALIDATION_ERROR_CODES` ;
- `LedgerBalanceValidationErrorCode` ;
- `LedgerBalanceValidationError` ;
- `LedgerBalanceValidationResult` ;
- `isLedgerBalanceValidationErrorCode(value)`.

La validation est pure, déterministe et sans accès à la base, au réseau ou à un fournisseur externe.

## 4. Codes d’erreur

Les codes stables sont :

- `INVALID_ACCOUNT_ID` : l’identifiant du compte est absent ;
- `CURRENCY_MISMATCH` : les composantes disponible et en attente n’utilisent pas la même devise ;
- `NEGATIVE_PENDING_AMOUNT` : le montant en attente est négatif ;
- `INVALID_AS_OF` : la date de calcul de la projection est invalide.

Ces codes doivent être journalisés côté backend sans exposer de données sensibles au client final.

## 5. Tests runtime ajoutés

Le fichier `packages/contracts/test/ledger-balance.test.mjs` couvre :

- une projection valide ;
- un solde disponible négatif explicitement accepté au niveau du contrat générique ;
- un identifiant de compte vide ;
- une divergence de devise ;
- un montant en attente négatif ;
- un horodatage invalide ;
- la reconnaissance des codes d’erreur connus et le rejet d’un code inconnu.

Le script du package contrats compile les sources TypeScript avant d’exécuter les tests Node, ce qui garantit que le module de validation doit d’abord être compilable.

## 6. Intégration backend attendue

Avant d’exposer ou persister une projection de solde, le backend doit :

1. charger le compte et vérifier son existence et son statut ;
2. reconstruire ou mettre à jour la projection dans la même logique transactionnelle que les écritures concernées ;
3. appeler `validateLedgerBalance` ;
4. refuser la publication d’une projection invalide ;
5. ouvrir une alerte d’intégrité si la projection calculée diverge du grand livre ;
6. conserver `asOf` afin de mesurer la fraîcheur de la projection.

## 7. Réconciliation

Une validation structurelle réussie ne prouve pas qu’un solde est comptablement juste. La réconciliation doit également comparer la projection à la somme des écritures du compte au même point de coupure.

Toute divergence doit :

- produire une métrique d’intégrité ;
- générer un événement auditable ;
- empêcher les opérations risquées lorsque le seuil défini est dépassé ;
- déclencher une reconstruction de projection ou une investigation selon la cause.

## 8. Éléments restant à construire

La prochaine couche doit encore ajouter :

- la persistance PostgreSQL des projections ;
- la séquence ou version de projection afin d’ordonner strictement les mises à jour ;
- la vérification que la devise de la projection correspond à celle du compte ;
- la reconstruction déterministe depuis les écritures ;
- les verrous ou mécanismes optimistes de concurrence ;
- les workers de réconciliation et de réparation ;
- l’exposition stable du contrat de validation aux consommateurs externes du package si nécessaire ;
- des tests d’intégration sur transactions concurrentes et reprise après incident.
