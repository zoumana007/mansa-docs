# Catalogue API — portefeuilles et soldes

## 1. Portée

Ce catalogue décrit les premières routes de consultation du domaine portefeuille. Les chemins, méthodes et types sont déclarés dans `mansa-platform/packages/contracts/src/wallet-api.ts`.

Préfixe : `/v1`.

Ces routes sont volontairement limitées à la lecture. Les mouvements financiers sont créés par les domaines paiement, transfert, dépôt, retrait ou service public afin de préserver les règles du grand livre.

## 2. Routes

### `GET /v1/wallets`

Retourne les portefeuilles accessibles à l’acteur authentifié.

Sortie : liste de `Wallet`.

Contraintes :

- un client ne voit que ses propres portefeuilles ;
- un employé ou administrateur ne voit que les portefeuilles autorisés par son périmètre ;
- les portefeuilles fermés peuvent être masqués par défaut mais restent consultables par les rôles habilités ;
- aucune donnée de grand livre interne n’est exposée.

### `GET /v1/wallets/:walletId`

Retourne les métadonnées d’un portefeuille.

Sortie : `Wallet`.

Le contrôle d’autorisation porte sur l’identifiant du portefeuille, le propriétaire, le pays et le périmètre administratif. Une ressource inaccessible ne doit pas révéler son existence.

### `GET /v1/wallets/:walletId/balance`

Retourne le solde courant d’un portefeuille.

Sortie : `WalletBalance` avec les montants `available`, `reserved`, `total` et l’horodatage `asOf`.

Règles :

- les montants utilisent le type `Money` et une devise unique ;
- `total` doit être cohérent avec les écritures comptables validées ;
- `available` tient compte des réservations et restrictions applicables ;
- la réponse ne doit jamais être recalculée à partir d’un historique incomplet côté client ;
- la précision monétaire dépend de la devise et aucun flottant n’est utilisé.

### `GET /v1/wallets/:walletId/transactions`

Retourne l’historique paginé du portefeuille.

Entrée : `ListTransactionHistoryQuery` dans les paramètres de requête.

Sortie : `TransactionHistoryPage`.

Filtres supportés par le contrat partagé : période, direction, type, statut, montant, recherche, tri et pagination. Les limites de pagination sont normalisées côté serveur.

## 3. Cohérence du solde

Le solde affiché est une projection du grand livre, pas une valeur indépendante modifiable directement. Toute correction passe par une écriture compensatrice autorisée et auditée.

Invariants minimaux :

1. Les montants d’un même solde ont la même devise que le portefeuille.
2. `available` et `reserved` ne sont jamais négatifs sauf produit explicitement autorisé.
3. Une transaction finalisée apparaît au plus une fois dans l’historique.
4. Une réservation diminue le disponible sans créer artificiellement de valeur.
5. Une annulation ou un remboursement produit une écriture traçable.
6. L’horodatage `asOf` permet au client de détecter une réponse ancienne.

## 4. Sécurité et confidentialité

- Authentification obligatoire.
- Autorisation par ressource et périmètre.
- Limitation de débit pour les consultations répétées.
- Masquage des données de contrepartie selon le rôle et le contexte.
- Journalisation des consultations administratives sensibles.
- Aucun numéro de carte complet, secret partenaire ou document KYC dans les réponses.
- Cache privé uniquement, avec invalidation appropriée après mouvement financier.

## 5. Cohérence technique

La source canonique des chemins et méthodes est :

- `WALLET_API_ROUTES`
- `WALLET_API_METHODS`
- `WalletApiContract`

Les contrôleurs NestJS et les clients mobiles ou web doivent réutiliser ces contrats. Les chemins ne doivent pas être dupliqués sous forme de chaînes dispersées.

## 6. Critères d’acceptation

- Chaque route documentée existe dans `WALLET_API_ROUTES` avec la même méthode.
- Les réponses utilisent `Wallet`, `WalletBalance` et `TransactionHistoryPage` du paquet de contrats.
- Un utilisateur ne peut pas consulter le portefeuille d’un autre utilisateur sans autorisation explicite.
- Tous les montants sont sérialisés sans perte de précision.
- Les soldes disponibles, réservés et totaux restent cohérents après paiement, remboursement et expiration d’une réservation.
- La pagination applique une limite maximale côté serveur.
- Les filtres invalides produisent une erreur API structurée.
- Une consultation administrative sensible est corrélée à un événement d’audit.
