# Inventaire, réservations et mouvements de stock

## 1. Objectif

Le module Inventaire fournit une source de vérité par commerce, établissement, produit et variante. Il couvre les quantités disponibles, les réservations liées aux commandes, les ajustements justifiés, les transferts entre établissements et l’historique complet des mouvements.

Il doit rester utilisable par une petite boutique tout en permettant une gestion multi-établissements. La gestion de stock est optionnelle par produit et configurable par commerce.

## 2. Modèle de quantité

Chaque article d’inventaire conserve :

- `quantityOnHand` : quantité physiquement comptabilisée ;
- `quantityReserved` : quantité temporairement affectée à des commandes ;
- quantité disponible : `quantityOnHand - quantityReserved` ;
- `reorderPoint` : seuil d’alerte facultatif ;
- `version` : version utilisée pour le verrouillage optimiste.

Toutes les quantités du contrat initial sont des entiers sûrs. Les produits vendus au poids ou au volume devront utiliser ultérieurement une unité de mesure et une précision explicites, jamais un flottant implicite.

Une quantité disponible négative est interdite sauf politique exceptionnelle, explicitement activée et auditée. Le comportement par défaut bloque la réservation ou la vente.

## 3. Statuts

Un article peut être :

- `ACTIVE` : disponible pour les opérations ;
- `INACTIVE` : temporairement désactivé ;
- `ARCHIVED` : conservé pour l’historique mais non réutilisable directement.

Une réservation peut être :

- `ACTIVE` : quantité réservée ;
- `COMMITTED` : vente confirmée et stock consommé ;
- `RELEASED` : réservation libérée ;
- `EXPIRED` : réservation libérée après expiration.

Une réservation terminale ne peut pas revenir à `ACTIVE`.

## 4. Mouvements

Tout changement de quantité produit un mouvement immuable. Les types initiaux sont :

- `RECEIPT` : réception de marchandises ;
- `SALE` : sortie liée à une vente ;
- `RETURN` : retour en stock ;
- `ADJUSTMENT_IN` et `ADJUSTMENT_OUT` : correction justifiée ;
- `TRANSFER_IN` et `TRANSFER_OUT` : transfert entre emplacements ;
- `RESERVATION` et `RESERVATION_RELEASE` : variation de la quantité réservée.

Chaque mouvement conserve l’article, la quantité, les soldes résultants, la référence métier, l’acteur, la clé d’idempotence, la date et le motif éventuel.

Un mouvement publié ne peut pas être supprimé ou réécrit. Une correction crée un nouveau mouvement compensatoire.

## 5. Réservation liée aux commandes

Quand une commande passe dans l’état configuré pour la réservation, le serveur :

1. verrouille les articles concernés ;
2. vérifie les quantités disponibles ;
3. crée une réservation unique liée à `orderId` ;
4. augmente les quantités réservées dans une même transaction ;
5. enregistre les mouvements et événements d’audit.

La confirmation de vente transforme la réservation en consommation de stock. L’annulation, l’échec du paiement ou l’expiration libère les quantités.

Une commande et une clé d’idempotence ne peuvent créer qu’une seule réservation active équivalente.

## 6. Ajustements

Un ajustement exige :

- une quantité entière non nulle ;
- un motif obligatoire ;
- la version attendue de l’article ;
- l’identité de l’acteur ;
- une clé d’idempotence.

Les rôles, seuils et obligations de double validation sont configurables. Un ajustement important ou répété peut déclencher une alerte fraude ou contrôle interne.

## 7. Transferts

Un transfert entre établissements crée atomiquement :

- un mouvement `TRANSFER_OUT` sur la source ;
- un mouvement `TRANSFER_IN` sur la destination ;
- une référence commune permettant la réconciliation.

Les deux articles doivent représenter le même produit et la même variante, appartenir au même commerce et utiliser des unités compatibles. Si l’une des deux écritures échoue, aucune ne doit être publiée.

Une évolution pourra introduire un état « en transit » pour les transferts physiques nécessitant expédition et réception séparées.

## 8. Mode hors ligne

Le mode hors ligne peut afficher un stock local indicatif et mettre en file des opérations non financières. Il ne garantit pas la disponibilité réelle en présence de plusieurs terminaux.

À la synchronisation :

- chaque commande et mouvement utilise un identifiant local unique et une clé d’idempotence ;
- le serveur applique le verrouillage et les règles finales ;
- un conflit de stock ne doit jamais être résolu silencieusement ;
- la vente est placée en exception ou suit une politique de stock négatif explicitement autorisée ;
- l’utilisateur reçoit une explication exploitable.

## 9. API initiale

Les contrats techniques se trouvent dans :

- `mansa-platform/packages/contracts/src/inventory.ts` ;
- `mansa-platform/packages/contracts/src/inventory-api.ts`.

Routes initiales :

- `POST /v1/merchant/inventory/items` ;
- `GET /v1/merchant/inventory/items` ;
- `GET /v1/merchant/inventory/items/:inventoryItemId` ;
- `POST /v1/merchant/inventory/items/:inventoryItemId/adjustments` ;
- `POST /v1/merchant/inventory/transfers` ;
- `GET /v1/merchant/inventory/movements` ;
- `POST /v1/merchant/inventory/reservations` ;
- `GET /v1/merchant/inventory/reservations/:reservationId` ;
- `POST /v1/merchant/inventory/reservations/:reservationId/status`.

Les listes sont paginées. Les filtres restent limités au commerce et aux établissements autorisés pour l’acteur.

## 10. Autorisations minimales

- un caissier consulte le stock utile à la vente dans son établissement ;
- un responsable peut effectuer des ajustements dans ses limites ;
- un gestionnaire de stock peut réceptionner, transférer et inventorier ;
- un propriétaire consulte tous ses établissements ;
- un administrateur Mansa n’intervient qu’avec un périmètre explicite, une justification et une trace d’audit.

Les coûts d’achat, marges et informations fournisseurs peuvent nécessiter des permissions distinctes de la simple consultation des quantités.

## 11. Alertes et reporting

Le module doit permettre :

- alertes de stock faible ou épuisé ;
- détection des réservations expirées ;
- rapprochement ventes–sorties de stock ;
- inventaire théorique comparé au comptage physique ;
- analyse des ajustements, pertes, retours et transferts ;
- export contrôlé par établissement et période.

Les alertes ne doivent pas exposer d’informations commerciales à un acteur non autorisé.

## 12. Critères d’acceptation

1. Une quantité non entière, négative lorsqu’elle doit être positive, ou hors plage sûre est rejetée.
2. Une réservation supérieure à la quantité disponible est rejetée par défaut.
3. Deux appels avec la même clé d’idempotence ne produisent pas deux mouvements.
4. Un conflit de version ne modifie aucun solde.
5. Un transfert est atomique et produit deux mouvements corrélés.
6. Une réservation libérée, expirée ou confirmée ne peut pas être retraitée.
7. Tout ajustement contient un motif et un acteur.
8. Les soldes résultants d’un mouvement correspondent à l’état persistant.
9. Les accès sont isolés par commerce et établissement.
10. Les opérations sensibles sont auditées et peuvent exiger une double validation.

## 13. Éléments restant à construire

- schéma PostgreSQL, contraintes et index ;
- services NestJS et transactions atomiques ;
- orchestration avec commandes, paiements et remboursements ;
- écrans d’inventaire, réception, comptage et transfert ;
- synchronisation hors ligne et gestion explicite des conflits ;
- alertes de seuil et tableaux de bord ;
- prise en charge des unités fractionnaires ;
- tests de concurrence, idempotence, sécurité et reprise.
