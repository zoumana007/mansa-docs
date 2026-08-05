# Portefeuilles, soldes et historique

## 1. Objet

Ce chapitre définit le comportement attendu des portefeuilles Mansa dans l’application Client et dans les interfaces d’administration. Il aligne la documentation fonctionnelle avec les contrats partagés du dépôt `mansa-platform`.

## 2. Modèle de portefeuille

Un portefeuille appartient à un utilisateur et possède au minimum :

- un identifiant stable ;
- un propriétaire ;
- un pays ;
- une devise ;
- un statut ;
- des dates de création et de dernière mise à jour.

Un même utilisateur peut posséder plusieurs portefeuilles lorsque la configuration pays, devise ou produit l’autorise. Les portefeuilles ne doivent jamais être fusionnés implicitement.

## 3. Statuts

Les statuts autorisés sont :

- `ACTIVE` : fonctionnement normal ;
- `RESTRICTED` : certaines opérations sont interdites ou limitées ;
- `SUSPENDED` : aucune nouvelle opération sortante n’est autorisée, sauf traitement explicitement prévu par la conformité ;
- `CLOSED` : portefeuille définitivement fermé aux nouvelles opérations.

Tout changement de statut doit contenir un motif, être autorisé, audité et, pour les actions sensibles, soumis aux règles de double validation. Le contrat peut inclure le statut courant attendu afin d’éviter d’écraser une modification concurrente.

## 4. Soldes

Le solde exposé comporte :

- `available` : montant immédiatement utilisable ;
- `reserved` : montant réservé pour des opérations non finalisées ;
- `total` : somme comptable exposée à la date indiquée ;
- `asOf` : horodatage de calcul.

Tous les montants sont exprimés en unités mineures entières. La devise des trois montants doit correspondre à celle du portefeuille. Une lecture de solde ne remplace jamais le grand livre comme source comptable de vérité.

## 5. Historique

L’historique d’un portefeuille est paginé et filtrable selon les contrats transverses de transaction. Chaque ligne doit exposer au minimum :

- l’identifiant et la référence de transaction ;
- le type, la direction et le statut ;
- le montant et la devise ;
- la contrepartie lorsque disponible ;
- les dates métier et techniques ;
- un lien ou identifiant de reçu lorsque disponible.

L’ordre par défaut est antéchronologique. Les filtres et tris doivent être appliqués côté serveur. Aucun écran client ne doit recalculer un solde comptable en additionnant l’historique reçu.

## 6. Contrat API partagé

Le contrat technique est défini dans `packages/contracts/src/wallet-api.ts`.

| Opération | Méthode | Route |
|---|---|---|
| Lister les portefeuilles | `GET` | `/v1/wallets` |
| Consulter un portefeuille | `GET` | `/v1/wallets/:walletId` |
| Consulter le solde | `GET` | `/v1/wallets/:walletId/balance` |
| Consulter l’historique | `GET` | `/v1/wallets/:walletId/transactions` |
| Modifier le statut | `PATCH` | `/v1/wallets/:walletId/status` |

La liste est paginée et peut être filtrée par propriétaire, pays, devise et statut.

## 7. Autorisations

- Un client ne peut consulter que ses propres portefeuilles, sauf délégation explicitement configurée.
- Un employé commerçant ne peut pas consulter les portefeuilles personnels d’un client.
- Les agents support disposent d’une vue limitée et masquée.
- Les rôles conformité et risque peuvent appliquer une restriction selon leurs permissions.
- La fermeture définitive et certaines suspensions nécessitent une approbation selon la matrice RBAC.

## 8. Critères d’acceptation

1. La liste des portefeuilles est paginée et les filtres ne permettent pas de contourner l’autorisation propriétaire.
2. Les montants sont toujours des entiers en unités mineures et conservent leur devise.
3. Une demande portant sur un portefeuille inaccessible ne révèle aucune donnée sensible.
4. Le changement de statut exige un motif non vide et produit un événement d’audit.
5. Une modification concurrente est refusée lorsque `expectedCurrentStatus` ne correspond plus au statut enregistré.
6. L’historique reste cohérent avec les écritures du ledger et utilise les conventions communes de pagination.
7. Les réponses et journaux ne contiennent aucun secret, numéro de carte complet ni document KYC.

## 9. Éléments à implémenter ensuite

- persistance des portefeuilles et des soldes projetés ;
- contrôleur NestJS et politiques d’autorisation ;
- tests de contrat et d’intégration ;
- écrans mobile de sélection du portefeuille, solde et historique ;
- projection temps réel alimentée par les événements du ledger.
