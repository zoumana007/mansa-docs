# Catalogue produits et gestion des stocks

## Objectif

Ce module fournit une source commune pour les produits vendus dans l’application Commerçant, le TPE, les mini-sites et les parcours de paiement. Il couvre les catégories, produits, variantes, prix, codes-barres, disponibilité par point de vente et mouvements de stock.

## Modèle de catalogue

Un produit appartient à un commerçant et peut être rattaché à une catégorie. Il possède :

- un nom et une description ;
- un type `PHYSICAL`, `SERVICE` ou `DIGITAL` ;
- un état `DRAFT`, `ACTIVE`, `INACTIVE` ou `ARCHIVED` ;
- un prix de base exprimé avec le type monétaire partagé ;
- un SKU et éventuellement un code-barres ;
- une catégorie fiscale configurable ;
- des images référencées par identifiant d’actif ;
- zéro, une ou plusieurs variantes ;
- un mode de suivi du stock ;
- une liste facultative de points de vente autorisés.

Les variantes servent à représenter une taille, une couleur, une capacité ou toute autre déclinaison. Chaque variante possède son propre SKU, son prix et éventuellement son code-barres.

## Catégories

Les catégories sont propres à un commerçant et peuvent être hiérarchiques. Leur slug est unique dans le périmètre du commerçant. La désactivation d’une catégorie ne supprime pas les produits existants et ne modifie pas leur historique.

## Cycle de vie des produits

- `DRAFT` : produit incomplet ou non publié.
- `ACTIVE` : produit visible et vendable sur les canaux autorisés.
- `INACTIVE` : produit conservé mais temporairement indisponible.
- `ARCHIVED` : produit retiré du catalogue courant et conservé pour l’historique.

Un produit archivé ne doit jamais être supprimé lorsqu’il est référencé par une commande, un paiement, une facture, une promotion ou un mouvement de stock.

## Modes de stock

- `NONE` : aucun suivi de quantité, par exemple pour certains services.
- `FINITE` : quantité disponible contrôlée par point de vente.
- `UNLIMITED` : quantité considérée comme illimitée.

Pour un stock fini, les quantités `onHandQuantity`, `reservedQuantity` et `availableQuantity` sont des entiers sûrs. La quantité disponible est calculée comme la quantité physique diminuée des réservations.

## Mouvements de stock

Toute variation produit un mouvement immuable. Les types initiaux sont :

- stock initial ;
- achat ou réapprovisionnement ;
- vente ;
- retour ;
- ajustement entrant ou sortant ;
- transfert entre points de vente ;
- réservation ;
- libération de réservation.

Chaque mouvement indique le commerçant, le point de vente, le produit, la variante éventuelle, la quantité, l’acteur, la date et la référence métier associée.

## Réservations et ventes concurrentes

La réservation est atomique et idempotente. Deux ventes concurrentes ne doivent pas consommer la même unité disponible.

Le traitement attendu est :

1. vérifier que le produit est actif et disponible dans le point de vente ;
2. verrouiller ou sérialiser la ligne de stock concernée ;
3. vérifier la quantité disponible ;
4. enregistrer la réservation et son mouvement ;
5. confirmer la sortie lors de la vente ou libérer la réservation en cas d’échec ;
6. conserver la corrélation avec la commande et le paiement.

## Codes-barres et SKU

- Le SKU est unique dans le périmètre du commerçant.
- Le code-barres est normalisé avant comparaison.
- Un même code-barres ne peut pas identifier deux articles actifs du même commerçant.
- Le TPE et l’application Commerçant utilisent le même catalogue.
- Les codes-barres ne contiennent aucun secret et ne constituent pas une preuve de paiement.

## Prix, fiscalité et promotions

Le prix de référence est conservé dans le catalogue. Les remises et avantages sont calculés par le module de promotions sans écraser le prix de base.

La catégorie fiscale est une référence configurable. Les taux et règles fiscales applicables sont résolus par pays, date, type de produit et statut du commerçant. Ils ne doivent pas être codés en dur dans l’application mobile ou le TPE.

## Stock faible et réapprovisionnement

Un seuil de réapprovisionnement facultatif peut être défini par article et point de vente. Le franchissement du seuil produit un événement et peut déclencher :

- une notification au commerçant ;
- une alerte dans le tableau de bord ;
- un export ou une proposition de commande fournisseur.

## Import et export

Les imports en lot doivent fournir un rapport ligne par ligne. Une erreur sur un produit ne doit pas créer silencieusement un catalogue partiel incohérent. Les exports masquent les données sensibles et sont audités lorsqu’ils contiennent des informations commerciales non publiques.

## Sécurité et gouvernance

- Les rôles distinguent consultation, création, modification de prix, ajustement de stock et archivage.
- Un ajustement manuel exige un motif.
- Les changements de prix et ajustements de stock sont audités.
- L’administration peut bloquer un produit, une catégorie, un commerçant ou un canal.
- Les images passent par un stockage contrôlé avec validation du type et analyse de sécurité.
- Aucun secret partenaire ni donnée de carte n’est stocké dans le catalogue.

## Contrat technique

Le contrat partagé est défini dans `mansa-platform/packages/contracts/src/catalog.ts` et exposé par le sous-chemin `@mansa/contracts/catalog`.

Il couvre :

- les états et types de produits ;
- les catégories et variantes ;
- les soldes et mouvements de stock ;
- la création et la mise à jour d’un produit ;
- les ajustements et réservations idempotentes ;
- la validation des quantités et le calcul du disponible.

## Critères d’acceptation

1. Un produit inactif ou archivé n’est pas vendable.
2. Les prix utilisent une devise et des unités mineures entières.
3. Les SKU actifs sont uniques par commerçant.
4. Une réservation concurrente ne peut pas créer un stock disponible négatif.
5. Toute variation de stock produit un mouvement immuable.
6. Un ajustement manuel exige un acteur et un motif.
7. Une commande annulée libère sa réservation de manière idempotente.
8. Le catalogue utilisé par le TPE et l’application Commerçant reste cohérent.
9. Les promotions n’écrasent jamais le prix de référence.
10. Les actions sensibles et imports sont auditables.
