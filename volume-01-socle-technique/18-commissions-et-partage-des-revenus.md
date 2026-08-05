# Commissions et partage des revenus

## 1. Objectif

Le moteur de commissions répartit les revenus d’une opération entre la plateforme, les agents, les commerçants, les partenaires, l’État et les réseaux techniques. Il est distinct du moteur de frais : les frais déterminent ce que paie le client, le commerçant ou un sponsor ; les commissions déterminent comment le revenu disponible est réparti entre les bénéficiaires.

## 2. Principes

- Chaque règle est versionnée, datée et limitée à un pays, une devise, un type d’opération et éventuellement un canal.
- Les montants sont exprimés en unités monétaires mineures entières.
- Les pourcentages sont exprimés en points de base : `100` représente `1 %`.
- Une politique ne produit de quote que lorsqu’elle est `ACTIVE`.
- Les règles sont évaluées par ordre de priorité stable.
- Les minimums et maximums s’appliquent à chaque allocation, jamais au montant global sans règle explicite.
- Les commissions acquises sont liées à la transaction d’origine et peuvent être annulées en cas de remboursement, d’échec tardif ou de fraude.
- Aucun paiement à un bénéficiaire n’est considéré comme effectué sans référence de règlement.

## 3. Bénéficiaires pris en charge

- `PLATFORM` : revenu de Mansa.
- `AGENT` : commission d’un agent de dépôt, retrait ou acquisition.
- `MERCHANT` : prime ou rétrocommission commerçant.
- `PARTNER` : banque, opérateur Mobile Money, agrégateur ou autre partenaire.
- `STATE` : quote-part contractuelle d’un service public.
- `NETWORK` : coût ou rémunération d’un réseau de paiement.

Un bénéficiaire générique peut être identifié uniquement par son type. Une règle destinée à une entité précise doit aussi contenir `beneficiaryId`.

## 4. Méthodes de calcul

Chaque règle utilise l’une des méthodes suivantes :

- `FIXED` : montant fixe par opération ;
- `PERCENTAGE` : pourcentage du montant de base ;
- `FIXED_PLUS_PERCENTAGE` : combinaison des deux.

Exemple sur une opération de `300 000 FCFA` avec une commission globale contractuelle de `1 %` : la quote globale vaut `3 000 FCFA`. La répartition peut ensuite attribuer, par exemple, `1 800 FCFA` à l’agent et `1 200 FCFA` à la plateforme. Cette répartition doit être modélisée par des règles explicites ; elle ne doit jamais être codée en dur.

## 5. Cycle de vie

### Politique

`DRAFT → ACTIVE → SUSPENDED → RETIRED`

Une politique retirée reste consultable pour expliquer les calculs historiques. Modifier une politique active ne doit pas réécrire les quotes et acquisitions déjà produites ; une nouvelle version doit être créée.

### Acquisition

`PENDING → ACCRUED → PAID`

Une acquisition `PENDING` ou `ACCRUED` peut passer à `REVERSED` lorsqu’une opération sous-jacente est annulée ou remboursée. Une acquisition payée nécessite une écriture de compensation séparée plutôt qu’une suppression.

## 6. Contrats techniques

Le dépôt `mansa-platform` expose :

- `packages/contracts/src/commission.ts` pour les politiques, règles, quotes, allocations et acquisitions ;
- `packages/contracts/src/commission-api.ts` pour les routes de gestion, simulation, consultation, annulation et marquage payé ;
- les sous-chemins de paquet `@mansa/contracts/commission` et `@mansa/contracts/commission-api`.

Routes principales :

- `GET|POST /v1/commission-policies` ;
- `GET|PATCH /v1/commission-policies/:policyId` ;
- `POST /v1/commission-policies/:policyId/quote` ;
- `GET /v1/commission-accruals` ;
- `GET /v1/commission-accruals/:accrualId` ;
- `POST /v1/commission-accruals/:accrualId/reverse` ;
- `POST /v1/commission-accruals/:accrualId/mark-paid`.

## 7. Contrôles administratifs

- La création et l’activation d’une politique doivent être séparées lorsque le risque financier est significatif.
- Une politique active doit passer les contrôles de chevauchement, devise, période, canal et type d’opération.
- Les changements sont soumis au RBAC, à la séparation des tâches et au journal d’audit.
- Une simulation doit afficher le montant de base, chaque allocation, le total distribué et la règle appliquée.
- Une alerte est déclenchée lorsque le total des commissions dépasse le revenu ou le plafond autorisé pour l’opération.

## 8. Comptabilité et rapprochement

Chaque acquisition doit produire ou référencer des écritures du grand livre. Les soldes de commissions à payer sont rapprochés avec les transactions, les remboursements et les règlements partenaires. Les exports financiers doivent permettre une agrégation par bénéficiaire, période, pays, devise, type d’opération, canal et statut.

## 9. Critères d’acceptation

1. Une politique inactive ne peut pas calculer de quote.
2. Une devise différente du montant de base est rejetée.
3. Les montants négatifs et pourcentages non entiers sont rejetés.
4. Les résultats sont déterministes pour une politique et un montant identiques.
5. Le total correspond exactement à la somme des allocations.
6. Une annulation conserve la trace de l’acquisition initiale.
7. Le marquage payé exige une référence de règlement et une date.
8. Les contrats documentés correspondent aux fichiers et routes du dépôt plateforme.
