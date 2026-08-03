# Catalogue API — paiements et demandes de paiement

## 1. Portée

Ce catalogue décrit les premières routes publiques du domaine paiement. Les chemins, méthodes et types partagés sont déclarés dans `mansa-platform/packages/contracts/src/payment-api.ts`.

Préfixe : `/v1`.

Toutes les créations financières exigent une clé d’idempotence. Les contrôleurs ne doivent jamais créer directement des écritures comptables : ils transmettent une commande au service métier, qui applique limites, frais, routage, autorisations et grand livre.

## 2. Routes

### `POST /v1/payments`

Crée un paiement à partir de `CreatePaymentCommand` et retourne un `Payment`.

Règles minimales :

- authentifier le payeur et vérifier l’accès au portefeuille source ;
- valider le bénéficiaire, la devise, le montant, les limites et le canal ;
- réutiliser la réponse d’une requête déjà traitée avec la même clé d’idempotence ;
- calculer les frais avant la confirmation finale ;
- ne jamais exposer de secret partenaire ou de donnée carte complète ;
- produire des événements d’audit et de corrélation.

### `GET /v1/payments/:paymentId`

Retourne un paiement accessible à l’acteur authentifié.

Une ressource appartenant à un autre périmètre ne doit pas révéler son existence. Les statuts exposés proviennent de `PAYMENT_STATUSES` et les erreurs utilisent le contrat d’erreur API commun.

### `POST /v1/payment-requests`

Crée une demande de paiement à partir de `CreatePaymentRequestCommand` et retourne un `PaymentRequest`.

La demande contient un montant, un bénéficiaire, une échéance éventuelle et les informations d’affichage nécessaires. Sa création ne déplace aucun fonds.

### `GET /v1/payment-requests/:paymentRequestId`

Retourne une demande de paiement visible par son créateur, son destinataire autorisé ou un rôle administratif habilité.

Les demandes expirées, annulées ou déjà réglées restent traçables mais ne peuvent pas être payées une seconde fois.

### `POST /v1/payment-requests/:paymentRequestId/pay`

Transforme une demande active en paiement. L’entrée utilise `PayPaymentRequestCommand`, contenant le portefeuille payeur et une clé d’idempotence. La sortie est un `Payment`.

L’opération doit verrouiller logiquement la demande afin d’empêcher deux règlements concurrents.

## 3. Idempotence et concurrence

1. La même clé et la même commande retournent le même résultat métier.
2. La même clé avec une commande différente est rejetée.
3. Deux paiements concurrents d’une même demande ne peuvent pas tous deux réussir.
4. Un timeout partenaire ne déclenche pas automatiquement un second débit.
5. Toute reprise utilise l’état de la tentative et la référence partenaire déjà enregistrée.

## 4. Cycle de vie

Le statut du paiement suit le contrat `PaymentStatus`. Les transitions sont pilotées par le service métier et les adaptateurs partenaires. Un paiement finalisé ne peut être modifié arbitrairement ; toute correction passe par remboursement, annulation autorisée ou écriture compensatrice traçable.

## 5. Sécurité

- Authentification obligatoire, sauf consultation publique explicitement conçue avec jeton limité.
- Autorisation par portefeuille, bénéficiaire, commerce, pays et rôle.
- Authentification renforcée selon montant, risque et canal.
- Limitation de débit et détection de répétitions anormales.
- Journalisation sans données sensibles.
- Validation stricte de tous les identifiants et montants.
- Vérification des webhooks partenaires par signature et anti-rejeu.

## 6. Cohérence technique

La source canonique est constituée de :

- `PAYMENT_API_ROUTES`
- `PAYMENT_API_METHODS`
- `PaymentApiContract`
- `PayPaymentRequestCommand`

Les contrôleurs NestJS et les clients doivent importer ces contrats au lieu de dupliquer les chemins et formes de requêtes.

## 7. Critères d’acceptation

- Chaque route documentée existe avec la même méthode dans le catalogue TypeScript.
- Toutes les créations financières utilisent une clé d’idempotence.
- Un portefeuille non autorisé ne peut pas être débité.
- Une demande ne peut être réglée qu’une seule fois.
- Les montants conservent leur précision et leur devise.
- Les frais et références sont conservés dans le résultat métier.
- Les erreurs sont structurées, corrélées et ne divulguent aucun secret.
- Les transitions finales restent immuables et toute correction est traçable.
