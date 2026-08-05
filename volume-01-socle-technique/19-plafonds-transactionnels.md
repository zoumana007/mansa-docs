# Plafonds transactionnels

## 1. Objectif

Les plafonds transactionnels protègent les utilisateurs, les commerçants, les partenaires et la plateforme contre les abus, les erreurs opérationnelles et les dépassements réglementaires. Ils s’appliquent avant toute autorisation financière et restent distincts des soldes disponibles, des frais, des commissions et des contrôles de fraude.

## 2. Portées prises en charge

Un plafond est rattaché à une portée et à un identifiant de portée :

- `USER` : utilisateur particulier ou professionnel ;
- `WALLET` : portefeuille déterminé ;
- `MERCHANT` : organisation commerçante ;
- `TERMINAL` : terminal de paiement physique ou logiciel ;
- `COUNTRY` : règle globale applicable à un pays.

Plusieurs plafonds peuvent s’appliquer à la même opération. L’autorisation n’est accordée que si chacun des plafonds applicables est respecté.

## 3. Périodes

Les périodes supportées sont :

- `PER_TRANSACTION` ;
- `DAILY` ;
- `WEEKLY` ;
- `MONTHLY`.

Les bornes de période sont calculées avec un fuseau horaire explicitement configuré par pays. Elles ne doivent jamais dépendre implicitement du fuseau du serveur.

## 4. Cycle de vie

Un plafond possède l’un des statuts suivants :

- `ACTIVE` : évalué pour les nouvelles opérations ;
- `SUSPENDED` : conservé mais non utilisable ;
- `EXPIRED` : période d’effet terminée.

Toute activation, suspension ou modification est soumise au RBAC, au journal d’audit et, pour les plafonds sensibles, à une double validation.

## 5. Évaluation

L’évaluation reçoit le montant demandé, la devise et l’instant de contrôle. Elle retourne :

- une décision `allowed` ;
- un motif : `WITHIN_LIMIT`, `LIMIT_EXCEEDED`, `CURRENCY_MISMATCH` ou `LIMIT_INACTIVE` ;
- le montant restant disponible en unité monétaire mineure.

Une opération de montant négatif est refusée. Une devise différente de celle du plafond est refusée. Le montant restant ne peut jamais devenir négatif.

## 6. Consommation et concurrence

La consommation d’un plafond contient le montant déjà utilisé, le nombre d’opérations et les bornes de la période courante. La réservation puis la consommation définitive doivent être atomiques afin d’éviter que deux paiements concurrents dépassent ensemble le plafond.

Le parcours cible est :

1. identifier tous les plafonds applicables ;
2. verrouiller ou réserver la capacité nécessaire ;
3. exécuter l’autorisation financière ;
4. confirmer la consommation si l’opération est acceptée ;
5. libérer la réservation en cas d’échec ou d’expiration ;
6. corriger la consommation par écriture compensatoire lors d’une annulation autorisée.

Aucune consommation historique ne doit être supprimée.

## 7. Hiérarchie des règles

Les plafonds réglementaires ou contractuels obligatoires priment sur les préférences utilisateur. Un utilisateur peut choisir un plafond personnel plus strict, mais jamais augmenter lui-même un plafond imposé par la conformité, la banque partenaire ou le pays.

En cas de chevauchement, la valeur la plus restrictive s’applique. La réponse interne doit conserver les identifiants des règles évaluées pour permettre l’audit et le support.

## 8. Contrats techniques

Le dépôt `mansa-platform` contient :

- `packages/contracts/src/transaction-limits.ts` pour les modèles, statuts, périodes et l’évaluation pure ;
- `packages/contracts/src/transaction-limits-api.ts` pour les routes de création, consultation, modification, évaluation, consommation, suspension et activation.

Routes principales :

- `GET|POST /v1/transaction-limits` ;
- `GET|PATCH /v1/transaction-limits/:limitId` ;
- `POST /v1/transaction-limits/:limitId/evaluate` ;
- `GET /v1/transaction-limits/:limitId/consumption` ;
- `POST /v1/transaction-limits/:limitId/suspend` ;
- `POST /v1/transaction-limits/:limitId/activate`.

## 9. Sécurité et administration

- Les changements de plafonds sont journalisés avec acteur, motif, ancienne valeur, nouvelle valeur et corrélation.
- Les demandes répétées utilisent une clé d’idempotence.
- Les réponses publiques n’exposent pas les règles internes de fraude ou de conformité.
- Une suspension d’urgence peut être propagée immédiatement sans redéploiement.
- Les limites par pays, partenaire et produit sont configurées par environnement.
- Aucun secret ou identifiant partenaire réel n’est stocké dans les contrats.

## 10. Critères d’acceptation

1. Un plafond inactif ou hors période refuse l’évaluation.
2. Une devise incompatible est refusée.
3. Une opération égale au montant restant est acceptée.
4. Une opération supérieure d’une unité mineure est refusée.
5. Deux opérations concurrentes ne peuvent pas consommer deux fois la même capacité.
6. Une suspension prend effet sans modifier l’historique.
7. Les plafonds personnels ne peuvent pas dépasser les plafonds obligatoires.
8. Toutes les décisions restent corrélables à la règle et à la période évaluées.
9. Les routes documentées correspondent aux contrats du dépôt plateforme.
