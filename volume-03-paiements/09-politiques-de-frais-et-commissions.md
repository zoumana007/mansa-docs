# Politiques de frais et commissions

## Objet

Les politiques de frais déterminent les commissions appliquées aux paiements, transferts, encaissements, retraits, règlements commerçants et services publics. Elles doivent être explicites, versionnées, testables et consultables avant confirmation par l’utilisateur ou le commerçant.

## Principes

- Tous les calculs utilisent les unités mineures de la devise et des entiers.
- Aucun pourcentage n’est calculé avec un nombre flottant.
- Une politique est liée à un pays, une devise, un type d’opération et, si nécessaire, un canal.
- La politique réellement utilisée est enregistrée avec la transaction afin de rendre le calcul reconstructible.
- Les frais affichés avant confirmation doivent être identiques aux frais comptabilisés, sauf nouvelle autorisation explicite.
- Les exonérations et subventions doivent être tracées comme des décisions métier, jamais comme des suppressions silencieuses.

## Méthodes de calcul

Le socle reconnaît :

- `FIXED` : montant fixe en unité mineure ;
- `PERCENTAGE` : pourcentage exprimé en points de base, où 10 000 points représentent 100 % ;
- `FIXED_PLUS_PERCENTAGE` : somme du montant fixe et du pourcentage.

Une politique peut également définir un minimum et un maximum. Le minimum est appliqué après le calcul brut, puis le maximum borne le résultat final.

## Répartition du paiement des frais

- `CUSTOMER` : le client paie les frais en plus du montant principal ;
- `MERCHANT` : les frais sont déduits du montant crédité au commerçant ;
- `SHARED` : les frais sont répartis entre le débiteur et le bénéficiaire ;
- `SPONSOR` : un tiers prend les frais en charge, sans modifier le débit client ni le crédit bénéficiaire.

En cas de partage d’un montant impair, la règle de répartition doit conserver chaque unité mineure. Le contrat actuel attribue la première moitié au client et le reliquat au bénéficiaire.

## États et validité

- `DRAFT` : préparation, non applicable aux opérations ;
- `ACTIVE` : utilisable par le moteur de tarification ;
- `SUSPENDED` : temporairement interdite ;
- `RETIRED` : définitivement retirée pour les nouvelles opérations.

Les dates d’effet doivent être évaluées avant le calcul par le service applicatif. Une transaction conserve l’identifiant et la version de la politique appliquée.

## Cycle de décision

1. Résoudre le pays, la devise, le type d’opération, le canal, le produit et le segment applicables.
2. Sélectionner une seule politique active à l’instant de la demande.
3. Calculer les frais avec des entiers.
4. Retourner un devis contenant montant principal, frais, total débité et net crédité.
5. Faire confirmer le devis lorsque l’interface l’exige.
6. Persister le devis et les écritures de grand livre dans la même opération atomique.

Une absence de politique ne doit pas être interprétée automatiquement comme des frais nuls. Le produit doit définir explicitement si l’opération est gratuite ou bloquée.

## Administration et gouvernance

Toute création ou modification d’une politique doit :

- utiliser une permission dédiée ;
- être journalisée avec auteur, motif et portée ;
- être soumise à double validation pour une modification sensible ;
- être versionnée sans modifier rétroactivement les transactions existantes ;
- être testée en Démo puis en Recette avant activation en Production ;
- permettre un retour contrôlé vers une version précédente.

Les commissions contractuelles des banques, opérateurs Mobile Money, réseaux de cartes et administrations doivent être séparées des frais facturés au client afin de calculer correctement la marge nette.

## Alignement avec le code

Le contrat de référence se trouve dans `packages/contracts/src/fee-policy.ts` du dépôt `mansa-platform`.

Il expose :

- les catalogues `FEE_CALCULATION_METHODS`, `FEE_PAYER_TYPES` et `FEE_POLICY_STATUSES` ;
- les types `FeePolicy` et `FeeQuote` ;
- la fonction `calculateFeeQuote` ;
- les garde-types associés.

Le sous-chemin public `@mansa/contracts/fee-policy` doit rester disponible pour les applications et services qui ne souhaitent pas importer l’ensemble du paquet.

## Critères d’acceptation

- Les calculs fixe, proportionnel et combiné sont exacts en unités mineures.
- Les minimums et maximums sont appliqués dans le bon ordre.
- Une devise différente est refusée.
- Une politique inactive est refusée.
- Une configuration négative ou incomplète est refusée.
- La somme des parts conserve toutes les unités mineures.
- Le devis expose le total débité et le net crédité selon le payeur.
- Les tests automatisés couvrent les principaux scénarios et erreurs.
