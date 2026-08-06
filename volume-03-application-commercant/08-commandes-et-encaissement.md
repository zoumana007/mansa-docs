# Commandes et encaissement commerçant

## Objectif

Ce module relie le catalogue, les stocks, les promotions et les paiements. Il fournit un modèle commun de commande pour l’application Commerçant, le TPE, le mini-site et les intégrations API.

## Canaux

Une commande indique son canal d’origine :

- `MERCHANT_APP` pour l’application Commerçant ;
- `TPE` pour une vente créée depuis un terminal ;
- `MINI_SITE` pour une commande passée sur la vitrine du commerçant ;
- `API` pour une intégration autorisée.

Le canal est conservé pour l’audit, les statistiques, les commissions et la gestion des incidents.

## Modes de remise

Les modes initiaux sont :

- `IN_STORE` : consommation ou retrait immédiat sur place ;
- `PICKUP` : retrait ultérieur ;
- `DELIVERY` : livraison ;
- `DIGITAL` : fourniture d’un service ou contenu numérique.

Les frais de livraison, frais de service et pourboires restent séparés du sous-total des articles.

## Cycle de vie

- `DRAFT` : panier modifiable, non engagé financièrement.
- `PENDING_PAYMENT` : prix figé, stock réservé et paiement attendu.
- `PAID` : paiement confirmé.
- `PREPARING` : commande en préparation.
- `READY` : commande prête à remettre.
- `COMPLETED` : commande terminée.
- `CANCELLED` : commande annulée avec motif et libération du stock.
- `REFUNDED` : commande remboursée selon les règles du module de remboursement.

Les transitions autorisées sont définies dans le contrat partagé. Une application cliente ne doit pas inventer une transition non acceptée par le backend.

## Lignes de commande

Chaque ligne conserve un instantané du produit au moment de la validation : nom, SKU, variante, quantité, prix unitaire, montant brut, remise, taxe et montant net.

La modification ultérieure du catalogue ne doit jamais modifier une commande historique. Les identifiants du produit et de la variante restent présents pour la traçabilité.

Les quantités sont des entiers sûrs strictement positifs. Les produits nécessitant des mesures fractionnaires devront utiliser ultérieurement une unité métier et une quantité en unité minimale entière.

## Calcul des montants

Le détail financier contient :

- sous-total ;
- total des remises ;
- total des taxes ;
- frais de service ;
- frais de livraison ;
- pourboire ;
- total général.

Tous les montants d’une commande utilisent la même devise. Les calculs sont effectués en unités mineures entières et vérifiés côté serveur. Le client ne constitue jamais la source de vérité du total à débiter.

## Promotions

Les promotions applicables sont évaluées avant la création de la demande de paiement. Les lignes conservent les identifiants des promotions appliquées et le montant exact de remise.

Une modification ou expiration de promotion après passage à `PENDING_PAYMENT` ne doit pas recalculer silencieusement la commande. Toute nouvelle évaluation exige une action explicite et une nouvelle version de prix.

## Réservation de stock

Pour les produits à stock fini :

1. le backend vérifie la disponibilité ;
2. le stock est réservé atomiquement ;
3. la commande conserve la référence de réservation ;
4. le paiement confirmé transforme la réservation en sortie de stock ;
5. l’échec, l’expiration ou l’annulation libère la réservation de manière idempotente.

Une commande ne peut pas passer à `PENDING_PAYMENT` si une ligne n’est plus disponible.

## Paiement et reçus

La commande référence le paiement sans dupliquer les données sensibles. Le statut `PAID` n’est obtenu qu’après confirmation autoritative du module Paiement.

Le reçu référence la commande, le paiement, le commerçant, le point de vente, les lignes, les montants et les taxes. Aucun numéro de carte complet, code secret ou donnée d’authentification ne doit apparaître sur le reçu.

## Annulation et remboursement

Une annulation exige un acteur, un motif et une clé d’idempotence. Les règles dépendent du statut :

- avant paiement, la commande peut être annulée et le stock libéré ;
- après paiement, l’annulation opérationnelle doit être accompagnée d’un remboursement ou d’une décision explicite ;
- une commande terminée n’est pas supprimée et passe par le module de remboursement ;
- les remboursements partiels devront être rattachés aux lignes et quantités concernées.

## Concurrence et versionnement

Les modifications de panier utilisent une version attendue. Le backend refuse une mise à jour fondée sur une version dépassée afin d’éviter qu’un terminal ou un employé n’écrase les changements d’un autre acteur.

La création et les changements d’état utilisent une clé d’idempotence. Une répétition réseau ne doit pas créer une deuxième commande, un deuxième paiement ou une double sortie de stock.

## Sécurité et autorisations

- Un employé ne voit et ne modifie que les commandes de son commerçant et des points de vente autorisés.
- Les remises manuelles, annulations et remboursements peuvent exiger une permission renforcée.
- Les changements de statut, prix forcés et motifs sont audités.
- Les données client sont minimisées et masquées dans les listes et journaux.
- Les notes ne doivent contenir aucun secret, numéro de carte ou document d’identité.

## Contrat technique

Le contrat partagé est défini dans `mansa-platform/packages/contracts/src/order.ts` et exposé par `@mansa/contracts/order`.

Il contient :

- les statuts, canaux et modes de remise ;
- le client facultatif et masqué ;
- les lignes et la ventilation financière ;
- la commande commerçant ;
- les commandes de création, modification et transition ;
- la validation des quantités, transitions et devises.

## Critères d’acceptation

1. Une commande créée deux fois avec la même clé d’idempotence ne produit qu’un seul résultat métier.
2. Une ligne possède une quantité entière strictement positive.
3. Tous les montants utilisent la devise de la commande.
4. Le total est recalculé et validé côté serveur.
5. Une commande en attente de paiement réserve son stock atomiquement.
6. Une annulation libère le stock une seule fois.
7. Le statut `PAID` exige une confirmation du module Paiement.
8. Une transition non autorisée est rejetée.
9. Une modification concurrente avec une version dépassée est rejetée.
10. L’historique conserve les prix, taxes, remises, acteurs et motifs sans exposer de données sensibles.
