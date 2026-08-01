# Application Commerçant — Points de vente

## 1. Objectif

Un commerce peut exploiter un ou plusieurs points de vente sans confondre leurs employés, terminaux, transactions et paramètres opérationnels. Chaque point de vente possède une identité propre et un cycle de vie contrôlé côté serveur.

## 2. Modèle de domaine

Le dépôt plateforme expose `MerchantLocation` dans `packages/domain/src/merchant-location.ts`.

Champs principaux :

- `id` : identifiant interne du point de vente ;
- `merchantId` : profil commerçant propriétaire ;
- `name` : nom affiché ;
- `countryCode` : code pays ISO 3166-1 alpha-2 ;
- `city` : ville d’exploitation ;
- `addressLine` : adresse opérationnelle ;
- `status` : état courant ;
- `statusReason` : motif de suspension ou fermeture ;
- `createdAt` : date de création.

Aucun secret de terminal, clé API, code PIN ou justificatif sensible n’est stocké dans cet agrégat.

## 3. Cycle de vie

| État | Description |
|---|---|
| `draft` | Point créé mais non autorisé à opérer |
| `active` | Encaissements et opérations autorisés selon la configuration |
| `suspended` | Exploitation temporairement bloquée avec motif |
| `closed` | Fermeture définitive et irréversible |

Transitions autorisées :

1. `draft` vers `active` après validation des informations requises.
2. `active` vers `suspended` avec motif obligatoire.
3. `suspended` vers `active` après levée du blocage.
4. Tout état non fermé vers `closed` avec motif obligatoire.

Un point fermé ne peut plus être modifié ni réactivé. Une réouverture commerciale doit créer un nouveau point de vente afin de préserver l’historique.

## 4. Portée opérationnelle

Le point de vente devient la portée de référence pour :

- les affectations d’employés ;
- les terminaux TPE et appareils autorisés ;
- les transactions et remboursements ;
- les horaires, limites et moyens de paiement ;
- les rapports, rapprochements et caisses ;
- les promotions et catalogues locaux.

Les lots suivants devront ajouter une relation explicite entre `MerchantStaffMember` et les points de vente autorisés, avec des limites par opération et par terminal.

## 5. Règles de sécurité

- Seul un point `active` peut initier ou accepter une opération financière.
- Le code pays est normalisé et validé au format ISO à deux lettres.
- Toute suspension, réactivation, fermeture ou modification importante est auditée.
- Les contrôles d’autorisation sont appliqués par l’API, jamais uniquement par l’interface.
- Une fermeture ne supprime aucune transaction historique.
- Les terminaux et employés perdent immédiatement leur capacité opérationnelle lorsque le point est suspendu ou fermé.
- Les données sensibles d’intégration restent dans des coffres de secrets ou des modules dédiés.

## 6. Critères d’acceptation

- Un point nouvellement créé est en état `draft` et ne peut pas opérer.
- Un point actif peut opérer.
- Un point suspendu ne peut pas opérer et conserve son motif.
- Une réactivation efface le motif de suspension.
- Un point fermé ne peut être ni modifié ni réactivé.
- Un nom, une ville, une adresse ou un identifiant vide est refusé.
- Un code pays autre que deux lettres est refusé.
- La sérialisation publique ne contient aucun secret.

## 7. Cohérence avec le code

L’agrégat est exporté par `packages/domain/src/index.ts` et couvert par `packages/domain/test/merchant-location.test.mjs`. Le prochain lot cohérent doit introduire les affectations d’employés aux points de vente et les portées de permission associées.
