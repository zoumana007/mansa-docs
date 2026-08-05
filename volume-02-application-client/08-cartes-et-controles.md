# Cartes, contrôles et remplacements

## Objectif

Le module Cartes permet au client de consulter ses cartes physiques et virtuelles, de choisir un produit éligible, de commander une carte, de régler ses contrôles d’usage et ses plafonds, puis de demander un remplacement. Aucun numéro de carte complet, cryptogramme ou code PIN ne doit être stocké dans les journaux, contrats publics ou outils d’administration.

## Types de cartes

- `PHYSICAL` : carte physique livrée au client.
- `VIRTUAL` : carte virtuelle durable.
- `VIRTUAL_TEMPORARY` : carte virtuelle à durée limitée.
- `VIRTUAL_DISPOSABLE` : carte virtuelle renouvelée après utilisation selon les règles du partenaire.

Les réseaux prévus par le contrat sont `VISA`, `MASTERCARD` et `LOCAL`. L’activation effective d’un réseau dépend d’un émetteur, d’un processeur et des autorisations contractuelles applicables.

## Produits de carte

Un produit de carte contient au minimum :

- un code métier stable et un nom affiché ;
- le type et le réseau ;
- les devises autorisées ;
- les frais d’émission et de remplacement ;
- les contrôles et plafonds par défaut ;
- un indicateur d’activation.

L’administration doit pouvoir désactiver un produit sans modifier les cartes déjà émises. Les frais applicables sont figés dans la transaction ou la demande concernée pour assurer l’auditabilité.

## Cycle de vie

États d’une carte : `PENDING`, `ACTIVE`, `FROZEN`, `BLOCKED`, `EXPIRED`, `CANCELLED`.

Transitions principales :

- `PENDING` vers `ACTIVE`, `BLOCKED` ou `CANCELLED` ;
- `ACTIVE` vers `FROZEN`, `BLOCKED`, `EXPIRED` ou `CANCELLED` ;
- `FROZEN` vers `ACTIVE`, `BLOCKED`, `EXPIRED` ou `CANCELLED` ;
- `BLOCKED` vers `CANCELLED` uniquement ;
- `EXPIRED` et `CANCELLED` sont définitifs.

Le gel est réversible. Le blocage est réservé aux situations de risque, perte, vol, conformité ou décision administrative autorisée. Toute transition doit contenir l’acteur, la raison, l’horodatage, le contexte d’authentification et l’identifiant d’audit.

## Contrôles d’usage

Chaque carte expose des interrupteurs configurables :

- paiement en magasin ;
- paiement en ligne ;
- sans contact ;
- retrait d’espèces ;
- usage international.

Le backend reste la source de vérité. L’application affiche immédiatement l’état demandé, puis confirme l’état réellement appliqué par le processeur. En cas d’échec, l’interface restaure l’état serveur et présente un message explicite.

## Plafonds

Les plafonds sont exprimés avec le type monétaire partagé et jamais en nombre flottant :

- paiement par transaction ;
- paiement quotidien ;
- retrait par transaction ;
- retrait quotidien ;
- paiement en ligne quotidien.

Les plafonds du client ne peuvent pas dépasser les limites du produit, du niveau KYC, du pays, du partenaire ou des politiques de risque. Une réduction peut être immédiate. Une hausse sensible peut exiger une authentification renforcée ou une approbation.

## Livraison

Une carte physique possède un suivi distinct : `PENDING`, `IN_PRODUCTION`, `SHIPPED`, `DELIVERED`, `FAILED` ou `CANCELLED`. Une carte virtuelle utilise `NOT_REQUIRED`.

Seules des références de transport et d’adresse sont conservées dans le contrat partagé. Les données détaillées d’adresse relèvent du service sécurisé dédié. Les références transporteur ne doivent pas permettre d’accéder à des informations personnelles sans autorisation.

## Remplacement

Motifs supportés : carte endommagée, perdue, volée, expirée ou autre motif justifié.

Une demande de remplacement :

1. utilise une clé d’idempotence ;
2. bloque ou annule l’ancienne carte selon le motif et la politique de risque ;
3. calcule et enregistre les frais applicables ;
4. crée une nouvelle carte seulement après validation ;
5. conserve le lien entre ancienne et nouvelle carte ;
6. ne copie jamais un secret de carte de l’ancien support.

États : `REQUESTED`, `APPROVED`, `ISSUED`, `REJECTED`, `CANCELLED`.

## API alignée avec le dépôt plateforme

Contrats maintenus dans :

- `packages/contracts/src/card.ts` ;
- `packages/contracts/src/card-api.ts`.

Routes principales :

- `GET /v1/card-products` ;
- `POST /v1/cards` ;
- `GET /v1/cards` ;
- `GET /v1/cards/:cardId` ;
- `PATCH /v1/cards/:cardId/status` ;
- `PATCH /v1/cards/:cardId/controls` ;
- `PATCH /v1/cards/:cardId/limits` ;
- `POST /v1/cards/:cardId/replacements` ;
- `GET /v1/cards/:cardId/replacements`.

Les commandes de création, changement d’état, modification des contrôles, modification des plafonds et remplacement sont idempotentes.

## Sécurité

- Authentification renforcée pour affichage de données sensibles, changement de plafond important et remplacement.
- Aucun PAN complet, PIN ou CVV dans les contrats, logs, notifications, analytics ou outils support.
- Masquage systématique avec les quatre derniers chiffres uniquement.
- Limitation de fréquence sur les commandes sensibles.
- Révocation immédiate des sessions suspectes selon la politique de risque.
- Journal d’audit obligatoire pour toute action client, agent, administrateur ou partenaire.

## Critères de recette

1. Une commande répétée avec la même clé d’idempotence ne crée pas deux cartes.
2. Une carte gelée refuse les autorisations selon la politique du processeur et peut être réactivée.
3. Une carte bloquée ne peut pas revenir à `ACTIVE`.
4. Une carte expirée ou annulée ne peut plus changer d’état.
5. Les contrôles visibles correspondent à l’état serveur après synchronisation.
6. Un plafond supérieur à la politique effective est rejeté avec un code d’erreur stable.
7. Une carte virtuelle ne demande aucune livraison.
8. Une demande de remplacement conserve le lien avec l’ancienne carte.
9. Les réponses et journaux n’exposent que les quatre derniers chiffres.
10. Toutes les mutations produisent un événement d’audit corrélable.
