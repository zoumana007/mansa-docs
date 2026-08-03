# Catalogue API — cartes

## 1. Portée

Ce catalogue décrit les routes du domaine cartes. Les chemins, méthodes et types partagés sont déclarés dans `mansa-platform/packages/contracts/src/card-api.ts` et `card.ts`.

Préfixe : `/v1`.

Le domaine couvre les cartes physiques, virtuelles, temporaires et jetables. La plateforme conserve uniquement les références et métadonnées nécessaires ; les données sensibles de carte doivent rester chez un processeur conforme et ne jamais être journalisées en clair.

## 2. Routes

### `POST /v1/cards`

Crée une carte à partir de `CreateCardCommand` et retourne un `CardReference`.

Règles minimales :

- exiger une clé d’idempotence ;
- vérifier que le portefeuille appartient à l’acteur et peut supporter le produit demandé ;
- vérifier le niveau KYC, le pays, la devise et les restrictions du produit ;
- exiger une adresse de livraison valide pour une carte physique ;
- appliquer les frais avant émission ;
- ne jamais retourner PAN complet, CVV, PIN ou clé de chiffrement ;
- conserver la référence du processeur dans un stockage protégé côté backend.

### `GET /v1/cards`

Liste les cartes accessibles à l’acteur. Les filtres supportés sont `walletId`, `status` et `type` via `ListCardsQuery`.

La réponse ne contient que des `CardReference`. Les cartes annulées ou expirées peuvent rester visibles dans l’historique selon la politique de rétention, sans possibilité de réactivation.

### `GET /v1/cards/:cardId`

Retourne une carte accessible à son propriétaire ou à un rôle administratif autorisé dans la bonne portée.

Une ressource hors portée ne doit pas révéler son existence. Le numéro est toujours masqué et limité aux quatre derniers chiffres.

### `POST /v1/cards/:cardId/status`

Change l’état d’une carte à partir de `ChangeCardStatusCommand`.

Les transitions autorisées sont pilotées par le backend. Le gel peut être réversible ; le blocage et l’annulation peuvent nécessiter une authentification renforcée ou une approbation selon le contexte. Une carte expirée ou annulée est finale.

### `PUT /v1/cards/:cardId/controls`

Remplace les contrôles d’usage à partir de `UpdateCardControlsCommand` : paiement en magasin, paiement en ligne, sans contact, retrait d’espèces et usage international.

Le backend doit valider la compatibilité avec le produit, le pays, le processeur et les règles de risque. Toute modification est auditée et propagée au processeur avant d’être considérée comme effective.

### `PUT /v1/cards/:cardId/limits`

Remplace les limites à partir de `UpdateCardLimitsCommand`.

Les limites client ne peuvent jamais dépasser les plafonds réglementaires, produit, pays ou risque. Les montants conservent leur devise et leur précision. Une diminution peut être immédiate ; une augmentation peut exiger une authentification renforcée ou une approbation.

## 3. Cycle de vie

Les statuts sont définis par `CARD_STATUSES` :

- `PENDING`
- `ACTIVE`
- `FROZEN`
- `BLOCKED`
- `EXPIRED`
- `CANCELLED`

Les états `EXPIRED` et `CANCELLED` sont finaux. Une réémission crée une nouvelle carte et une nouvelle référence au lieu de modifier silencieusement l’ancienne.

## 4. Sécurité

- Authentification obligatoire sur toutes les routes.
- Autorisation par propriétaire, portefeuille, pays, organisation et environnement.
- Authentification renforcée pour les actions sensibles selon le risque.
- Aucun PAN complet, CVV, PIN ou secret processeur dans les réponses, logs, événements ou analytics.
- Chiffrement des références sensibles et rotation des clés hors dépôt.
- Audit de chaque création, changement de statut, contrôle et limite.
- Protection contre le rejeu et idempotence des commandes mutantes.
- Blocage global ou par produit possible depuis l’administration.

## 5. Cohérence technique

La source canonique est constituée de :

- `CARD_API_ROUTES`
- `CARD_API_METHODS`
- `CardApiContract`
- `ListCardsQuery`
- `CreateCardCommand`
- `ChangeCardStatusCommand`
- `UpdateCardControlsCommand`
- `UpdateCardLimitsCommand`
- `CardReference`

Le paquet `@mansa/contracts` expose le catalogue via `@mansa/contracts/card-api`. Les contrôleurs NestJS et les clients mobiles ou web doivent importer ce contrat au lieu de dupliquer les routes.

## 6. Critères d’acceptation

- Chaque route documentée possède la même méthode et le même chemin dans le catalogue TypeScript.
- Toute création de carte est idempotente.
- Les données sensibles de carte ne sont jamais exposées ni journalisées.
- Une carte finale ne peut pas être réactivée.
- Les contrôles et limites sont validés côté serveur.
- Une limite client ne dépasse jamais un plafond supérieur applicable.
- Les changements sont audités avec acteur, contexte, ancienne valeur et nouvelle valeur.
- Les erreurs processeur sont corrélées sans divulguer de données sensibles.
- Les applications utilisent les types partagés du paquet de contrats.
