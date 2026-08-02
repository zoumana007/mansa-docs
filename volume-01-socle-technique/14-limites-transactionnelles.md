# Limites transactionnelles

## Objectif

Mansa applique des limites déterministes avant toute opération financière afin de protéger le client, respecter les règles KYC, réduire la fraude et permettre une configuration par pays, produit, canal et profil.

## Types de limites

Une politique peut définir :

- un montant maximal par opération ;
- un cumul maximal journalier ;
- un nombre maximal d’opérations journalier ;
- un cumul maximal mensuel ;
- un nombre maximal d’opérations mensuel.

Les montants sont toujours exprimés en unité monétaire mineure avec un entier. Les valeurs flottantes sont interdites.

## Portée

Les limites sont évaluées pour une combinaison explicite : pays, devise, type d’opération, canal, niveau KYC et, lorsque nécessaire, segment client ou partenaire.

Une politique plus spécifique remplace une politique générale uniquement pour les champs qu’elle définit. La version effectivement appliquée doit être conservée avec la décision.

## Décisions

Le moteur partagé retourne :

- `ALLOW` lorsque toutes les limites sont respectées ;
- `DENY_PER_TRANSACTION` lorsque le montant unitaire dépasse le plafond ;
- `DENY_DAILY_AMOUNT` lorsque le cumul journalier projeté dépasse le plafond ;
- `DENY_DAILY_COUNT` lorsque le nombre journalier projeté dépasse le plafond ;
- `DENY_MONTHLY_AMOUNT` lorsque le cumul mensuel projeté dépasse le plafond ;
- `DENY_MONTHLY_COUNT` lorsque le nombre mensuel projeté dépasse le plafond.

L’évaluation est faite sur les compteurs projetés, c’est-à-dire après ajout de l’opération demandée. En cas de plusieurs dépassements, l’ordre ci-dessus est stable afin de produire une réponse déterministe.

## Contrat technique partagé

Le paquet `packages/security` expose `evaluateTransactionLimits`. La fonction :

1. refuse les montants, cumuls ou compteurs négatifs ;
2. refuse les compteurs non entiers ;
3. vérifie la cohérence de la devise entre requête et politique ;
4. calcule les valeurs projetées sans accès réseau ni base de données ;
5. retourne la décision, la règle déclenchée et les valeurs projetées ;
6. ne modifie jamais les compteurs persistés.

La réservation atomique des limites appartient au service transactionnel. Celui-ci doit utiliser une transaction de base de données ou un mécanisme équivalent pour éviter deux autorisations concurrentes dépassant ensemble le plafond.

## Administration et gouvernance

Les limites sont configurables depuis l’administration avec : version, état, date d’effet, auteur, motif et double validation pour les hausses sensibles. Une baisse urgente peut être activée immédiatement selon les droits prévus, avec journal d’audit obligatoire.

Les valeurs de production ne doivent pas être codées en dur dans les applications clientes. Les applications peuvent afficher les limites retournées par l’API, mais le backend reste l’autorité.

## Traçabilité

Chaque décision persistée contient au minimum : identifiant d’opération, identifiant et version de politique, devise, montants projetés, compteurs projetés, décision, règle déclenchée, horodatage et identifiant de corrélation.

## Critères d’acceptation

1. Une opération exactement égale à une limite est autorisée.
2. Une opération dépassant une limite d’une unité est refusée.
3. Les cumuls sont évalués après ajout du montant demandé.
4. Les compteurs sont évalués après ajout d’une opération.
5. Les entrées négatives ou non entières sont refusées.
6. Une devise différente de celle de la politique est refusée.
7. La décision est déterministe lorsque plusieurs limites sont dépassées.
8. Les tests couvrent chaque type de plafond et les valeurs frontières.
