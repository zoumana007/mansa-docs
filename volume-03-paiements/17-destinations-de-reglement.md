# Destinations de règlement

## Objet

Une destination de règlement indique où les fonds dus à un commerçant doivent être versés après constitution et validation d’un lot. Elle est distincte de la planification, qui détermine quand les règlements sont générés.

## Types pris en charge

- `BANK_ACCOUNT` : compte bancaire référencé par un jeton de coffre-fort ;
- `MOBILE_MONEY` : compte Mobile Money référencé par un jeton fournisseur ;
- `INTERNAL_WALLET` : portefeuille interne Mansa.

Aucune coordonnée bancaire ou Mobile Money complète ne doit être stockée dans les contrats ou journaux. Seuls des jetons techniques et une référence masquée sont autorisés.

## États

- `PENDING_VERIFICATION` : destination créée mais non utilisable ;
- `ACTIVE` : destination vérifiée et utilisable pour un règlement ;
- `SUSPENDED` : utilisation temporairement bloquée ;
- `DISABLED` : destination désactivée.

Une destination en attente ne peut pas être activée par un simple changement d’état : elle doit passer par l’opération de vérification.

## Données obligatoires

Chaque destination contient :

- un identifiant unique ;
- le commerçant propriétaire ;
- le type de destination ;
- la devise et le pays ;
- exactement une référence technique parmi compte bancaire, Mobile Money ou portefeuille interne ;
- une référence masquée destinée à l’affichage ;
- un fournisseur facultatif ;
- le nom du titulaire facultatif ;
- l’indicateur de destination par défaut ;
- l’état et les horodatages.

## Règles métier

1. La devise est normalisée sur trois lettres majuscules.
2. Le pays est normalisé sur deux lettres majuscules.
3. Une seule référence technique est autorisée par destination.
4. Le type doit correspondre à la référence fournie.
5. Une référence masquée non vide est obligatoire.
6. Une destination nouvelle est toujours créée dans l’état `PENDING_VERIFICATION`.
7. La vérification produit l’état `ACTIVE` et renseigne `verifiedAt`.
8. La suspension ou la désactivation doit être auditée.
9. Le changement de destination par défaut doit être atomique : une seule destination par commerçant, devise et contexte peut être par défaut.
10. Un lot de règlement conserve l’identifiant de la destination utilisée, même si celle-ci est modifiée ou désactivée ensuite.

## Vérification attendue

Selon le partenaire, la vérification peut utiliser :

- un micro-virement bancaire ;
- une confirmation du titulaire via le partenaire ;
- une validation KYC/KYB et du nom de compte ;
- une validation d’un portefeuille interne déjà contrôlé.

La preuve détaillée reste dans le système sécurisé ou chez le partenaire. Le contrat partagé ne conserve que le statut et les horodatages nécessaires.

## Sécurité et administration

- Les jetons de destination proviennent d’un coffre-fort ou d’un adaptateur partenaire.
- Les valeurs complètes sont interdites dans les logs, événements, exports et messages d’erreur.
- L’ajout ou le remplacement d’une destination sensible peut nécessiter une authentification renforcée et une période de refroidissement.
- Les actions administratives doivent respecter la séparation des tâches et l’approbation à quatre yeux lorsque configurée.
- Une destination suspendue ne doit pas recevoir de nouveau lot.

## Alignement avec le code

Le contrat de référence est `packages/contracts/src/settlement-destination.ts` dans `mansa-platform`.

Il expose :

- `SETTLEMENT_DESTINATION_TYPES` ;
- `SETTLEMENT_DESTINATION_STATUSES` ;
- `SettlementDestination` et `SettlementDestinationDetails` ;
- les commandes de création, vérification et changement d’état ;
- `createSettlementDestination` ;
- `verifySettlementDestination` ;
- `changeSettlementDestinationStatus` ;
- les gardes de type associées ;
- le sous-chemin public `@mansa/contracts/settlement-destination`.

Les tests de référence se trouvent dans `packages/contracts/test/settlement-destination.test.mjs`.

## Critères d’acceptation

- Une destination valide est créée en attente de vérification.
- Les codes pays et devise sont normalisés et validés.
- Une destination ambiguë ou incompatible avec son type est rejetée.
- Une activation directe sans vérification est impossible.
- Une destination vérifiée peut être suspendue ou désactivée.
- Les références affichées sont masquées et aucune coordonnée réelle n’est versionnée.
- Le contrat est compilable, exporté et couvert par des tests automatisés.
