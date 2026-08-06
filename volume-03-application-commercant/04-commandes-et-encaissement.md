# Commandes, encaissement et exécution commerçant

## 1. Objectif

Le module Commandes permet à un commerçant de créer, modifier, encaisser et exécuter une commande depuis l’application Commerçant, l’application TPE, un mini-site ou une intégration API. Il s’applique aux restaurants, boutiques, services, commerces de proximité et vendeurs professionnels.

## 2. Canaux

Les canaux contractuels sont :

- `MERCHANT_APP` : application Commerçant ;
- `TPE` : terminal Android ;
- `MINI_SITE` : mini-site public du commerce ;
- `API` : intégration d’un logiciel tiers.

Le canal d’origine est conservé pendant toute la vie de la commande afin de permettre l’analyse, le support et la réconciliation.

## 3. Modes d’exécution

- `IN_STORE` : consommation ou remise sur place ;
- `PICKUP` : retrait ultérieur ;
- `DELIVERY` : livraison ;
- `DIGITAL` : service ou contenu numérique.

Chaque établissement peut activer ou désactiver les modes disponibles depuis l’administration commerçant.

## 4. Cycle de vie

Le cycle nominal est :

1. `DRAFT` : panier modifiable ;
2. `PENDING_PAYMENT` : montant figé en attente d’encaissement ;
3. `PAID` : paiement confirmé ;
4. `PREPARING` : préparation en cours lorsque nécessaire ;
5. `READY` : commande prête ;
6. `COMPLETED` : commande remise ou service terminé.

Les états terminaux sont `CANCELLED` et `REFUNDED`. Une commande publiée ne doit jamais être supprimée physiquement.

Transitions autorisées :

- `DRAFT` vers `PENDING_PAYMENT` ou `CANCELLED` ;
- `PENDING_PAYMENT` vers `PAID` ou `CANCELLED` ;
- `PAID` vers `PREPARING`, `READY`, `COMPLETED` ou `REFUNDED` ;
- `PREPARING` vers `READY`, `CANCELLED` ou `REFUNDED` ;
- `READY` vers `COMPLETED`, `CANCELLED` ou `REFUNDED` ;
- `COMPLETED` vers `REFUNDED`.

Toute transition refusée retourne une erreur métier stable et produit une trace d’audit.

## 5. Calcul des montants

Une ligne contient le prix unitaire, le montant brut, la remise, la taxe et le montant net. La commande contient :

- sous-total ;
- total des remises ;
- total des taxes ;
- frais de service ;
- frais de livraison ;
- pourboire ;
- total général.

Tous les montants sont exprimés en unités mineures entières. Une commande ne peut contenir qu’une seule devise. Le serveur recalcule toujours les montants à partir du catalogue, des règles fiscales et des promotions ; il ne fait jamais confiance aux totaux envoyés par le terminal ou l’application.

## 6. Paiement

Le passage à `PENDING_PAYMENT` réserve les prix et, si la gestion de stock est active, les quantités. Le paiement peut être réalisé par carte, portefeuille Mansa, QR, Mobile Money, espèces déclarées ou autre moyen autorisé pour le commerce.

Le passage à `PAID` exige une confirmation provenant du module Paiements. Une simple réponse locale du TPE ne suffit pas. Le lien `paymentId` permet la réconciliation entre commande, paiement, reçu et écritures comptables.

Une même clé d’idempotence ne peut créer qu’une seule commande ou une seule transition financière.

## 7. Annulation et remboursement

Avant paiement, l’annulation libère les réservations de stock. Après paiement, une annulation qui implique un retour d’argent doit déclencher le module de remboursement et ne passe à `REFUNDED` qu’après confirmation financière.

Chaque annulation ou remboursement conserve :

- l’acteur ;
- la date ;
- le motif ;
- le montant ;
- la référence du paiement et du remboursement ;
- la décision d’autorisation éventuelle.

Les seuils, délais et besoins de double validation sont configurables.

## 8. Fonctionnement hors ligne

Le mode hors ligne peut permettre la constitution du panier et l’impression d’un ticket provisoire. Il ne doit pas déclarer un paiement en ligne confirmé sans preuve cryptographique ou autorisation différée du fournisseur.

Les commandes créées hors ligne utilisent des identifiants locaux uniques, une heure locale, un numéro de terminal et une clé d’idempotence. La synchronisation serveur détecte les doublons et applique les règles de conflit.

## 9. Contrat API initial

Le contrat technique est défini dans `mansa-platform/packages/contracts/src/order-api.ts`.

Routes initiales :

- `POST /v1/merchant/orders` ;
- `GET /v1/merchant/orders` ;
- `GET /v1/merchant/orders/:orderId` ;
- `PATCH /v1/merchant/orders/:orderId` ;
- `POST /v1/merchant/orders/:orderId/status`.

La liste est paginée et filtrable par commerce, établissement, état, canal, mode d’exécution, client, paiement et période.

## 10. Autorisations minimales

- le caissier peut créer et encaisser dans son établissement ;
- le préparateur peut consulter et faire progresser les commandes qui lui sont accessibles ;
- le responsable peut annuler selon les seuils configurés ;
- le propriétaire peut consulter tous ses établissements ;
- un administrateur Mansa ne peut intervenir que selon son périmètre et avec audit ;
- une opération sensible peut exiger une authentification renforcée et une double validation.

## 11. Critères d’acceptation

1. Une quantité nulle, négative ou non entière est rejetée.
2. Une commande multidevise est rejetée.
3. Une transition non autorisée est rejetée sans modifier l’état.
4. Deux requêtes avec la même clé d’idempotence ne créent pas deux commandes.
5. Le total serveur correspond exactement aux lignes et frais applicables.
6. Une commande ne devient `PAID` qu’après confirmation du paiement.
7. Une annulation après paiement ne contourne pas le remboursement.
8. Les données client exposées sont masquées selon le rôle.
9. Chaque changement d’état est corrélé et audité.
10. Les filtres d’établissement empêchent tout accès transversal non autorisé.

## 12. Éléments restant à construire

- contrôleurs et services NestJS ;
- persistance PostgreSQL et verrouillage optimiste ;
- réservation et décrément de stock ;
- orchestration paiement, remboursement et reçu ;
- synchronisation hors ligne ;
- écrans Commerçant et TPE ;
- tests unitaires, d’intégration, sécurité et reprise.
