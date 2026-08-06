# Cycle de vie des portefeuilles

## Objet

Ce document définit le contrat minimal des portefeuilles Mansa, de leurs soldes et des réservations de fonds. Il s’applique aux applications Client, Commerçant, TPE, Admin Lite et au portail d’administration.

## Principes comptables

- Un portefeuille appartient à un seul propriétaire, un pays et une devise.
- Tous les montants sont exprimés en unités mineures entières.
- Le solde affiché est une projection du grand livre en partie double ; il ne constitue pas une source de vérité indépendante.
- `total = available + reserved` doit toujours être vrai pour une même devise.
- Chaque réponse de solde expose la séquence du ledger et la date de calcul.
- Une opération financière ne modifie jamais directement un solde sans écriture équilibrée dans le grand livre.

## États

| État | Signification |
|---|---|
| `PENDING_ACTIVATION` | portefeuille créé mais non encore utilisable |
| `ACTIVE` | opérations autorisées selon les droits et plafonds |
| `RESTRICTED` | seules les opérations explicitement autorisées restent possibles |
| `SUSPENDED` | toute nouvelle opération financière est bloquée |
| `CLOSED` | état final, aucune réouverture automatique |

Transitions autorisées :

- `PENDING_ACTIVATION` vers `ACTIVE`, `RESTRICTED` ou `CLOSED` ;
- `ACTIVE` vers `RESTRICTED`, `SUSPENDED` ou `CLOSED` ;
- `RESTRICTED` vers `ACTIVE`, `SUSPENDED` ou `CLOSED` ;
- `SUSPENDED` vers `ACTIVE`, `RESTRICTED` ou `CLOSED` ;
- `CLOSED` ne possède aucune transition sortante.

Chaque changement d’état exige un motif, un acteur, la version attendue et une clé d’idempotence. Une fermeture doit être refusée tant que le solde n’est pas nul, qu’une réservation active existe ou qu’un litige impose la conservation du portefeuille.

## Réservations de fonds

Une réservation immobilise une partie du solde disponible sans finaliser immédiatement une écriture de dépense. Les motifs normalisés sont : autorisation de paiement, transfert en attente, retrait en attente, litige, revue conformité ou revue manuelle.

Une réservation possède un identifiant, un montant, une référence métier, un statut et éventuellement une expiration. Les statuts sont `ACTIVE`, `RELEASED`, `CAPTURED` et `EXPIRED`.

Règles obligatoires :

1. La création est idempotente par référence métier et clé d’idempotence.
2. Le montant doit être strictement positif et dans la devise du portefeuille.
3. Le montant réservé ne peut dépasser le solde disponible.
4. Une réservation finalisée ne peut plus être libérée ou capturée une seconde fois.
5. L’expiration est traitée par un worker idempotent et produit un événement d’audit.
6. Une capture doit être corrélée à l’écriture définitive du ledger.

## Surface API

| Opération | Méthode | Route |
|---|---|---|
| Créer un portefeuille | `POST` | `/v1/wallets` |
| Lister les portefeuilles | `GET` | `/v1/wallets` |
| Consulter un portefeuille | `GET` | `/v1/wallets/:walletId` |
| Consulter le solde | `GET` | `/v1/wallets/:walletId/balance` |
| Consulter l’historique | `GET` | `/v1/wallets/:walletId/transactions` |
| Lister les réservations | `GET` | `/v1/wallets/:walletId/holds` |
| Créer une réservation | `POST` | `/v1/wallets/:walletId/holds` |
| Libérer une réservation | `POST` | `/v1/wallets/:walletId/holds/:holdId/release` |
| Changer l’état | `POST` | `/v1/admin/wallets/:walletId/status` |

Les routes de lecture vérifient la propriété ou un périmètre administratif explicite. Les routes d’écriture utilisent la corrélation, l’idempotence et l’audit décrits dans les conventions API.

## Autorisations et conformité

- Un client consulte uniquement ses propres portefeuilles, sauf mandat explicite.
- Un employé commerçant est limité à son organisation et à ses permissions.
- Un administrateur est limité par pays, environnement et rôle.
- La suspension, la restriction et la fermeture sont des actions sensibles auditées.
- Les décisions de conformité ne doivent pas exposer au client les règles internes de détection de fraude.
- Les données de solde et d’historique ne doivent jamais apparaître en clair dans les journaux techniques.

## Critères d’acceptation

- Une transition interdite est rejetée sans modification persistante.
- Deux créations avec la même clé d’idempotence retournent le même portefeuille.
- Une version obsolète retourne une erreur de concurrence explicite.
- Une réservation réduit `available` et augmente `reserved` du même montant sans modifier `total`.
- Une libération restaure exactement le montant disponible une seule fois.
- Une capture produit des écritures de ledger équilibrées et une corrélation vérifiable.
- Un portefeuille fermé ne peut recevoir aucune nouvelle opération.
- Un acteur hors périmètre ne peut lire ni modifier le portefeuille.
- Les contrats TypeScript de référence se trouvent dans `packages/contracts/src/wallet.ts` et `packages/contracts/src/wallet-api.ts`.
