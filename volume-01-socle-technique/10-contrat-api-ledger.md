# Contrat API interne du grand livre

## 1. Objet

Ce document fixe le contrat d’intégration entre les modules métier Mansa et le grand livre. Le grand livre reste un service interne : les applications Client, Commerçant, TPE et les intégrations partenaires ne doivent jamais publier directement des écritures comptables.

Le contrat TypeScript de référence se trouve dans :

- `mansa-platform/packages/contracts/src/ledger.ts` pour les modèles et invariants ;
- `mansa-platform/packages/contracts/src/ledger-api.ts` pour les routes et méthodes internes.

La stratégie de persistance PostgreSQL correspondante est détaillée dans `11-persistance-ledger-postgresql.md`.

## 2. Routes de référence

| Opération | Méthode | Route |
| --- | --- | --- |
| Publier une transaction | `POST` | `/v1/internal/ledger/transactions` |
| Lire une transaction | `GET` | `/v1/internal/ledger/transactions/:transactionId` |
| Compenser une transaction | `POST` | `/v1/internal/ledger/transactions/:transactionId/reverse` |
| Lire un compte | `GET` | `/v1/internal/ledger/accounts/:accountId` |
| Lire un solde projeté | `GET` | `/v1/internal/ledger/accounts/:accountId/balance` |
| Lister les écritures | `GET` | `/v1/internal/ledger/accounts/:accountId/entries` |

Ces routes sont réservées aux services autorisés. Elles ne doivent pas être exposées telles quelles au réseau public.

### 2.1 Authentification service-à-service transitoire

Les routes ledger déjà exposées par l’API gateway sont protégées par `InternalServiceGuard`.

Le mécanisme actuel est volontairement transitoire :

- le service appelant transmet `x-mansa-internal-token` ;
- la valeur attendue provient exclusivement de `INTERNAL_SERVICE_TOKEN` dans l’environnement d’exécution ;
- aucun jeton réel ne doit être versionné ;
- une configuration absente ou inférieure à 32 caractères bloque les appels en mode fail-closed ;
- une valeur erronée retourne une erreur d’autorisation sans révéler le secret attendu ;
- la comparaison du jeton utilise une comparaison en temps constant lorsque les longueurs sont identiques.

Ce mécanisme doit pouvoir être remplacé ultérieurement par mTLS ou une identité de workload signée sans modifier le contrat métier des contrôleurs. En production, la valeur doit provenir d’un gestionnaire de secrets et être rotatable.

Le comportement du guard est couvert dans `mansa-platform/apps/api-gateway/test/internal-service.guard.test.mjs` : absence de configuration, jeton trop court, en-tête absent, jeton invalide, correspondance exacte et en-tête multi-valeur.

## 3. Publication d’une transaction

La commande de publication contient au minimum :

- une référence métier stable ;
- un type de transaction ;
- au moins deux écritures ;
- une clé d’idempotence ;
- un identifiant de corrélation ;
- un code pays ;
- une date d’occurrence ;
- éventuellement des métadonnées non sensibles.

Chaque écriture indique un compte, un sens `DEBIT` ou `CREDIT`, un montant en unités mineures et éventuellement une description.

Avant toute persistance, le backend doit appeler la validation partagée et refuser la publication si :

- il y a moins de deux écritures ;
- un montant est nul ou négatif ;
- plusieurs devises sont présentes ;
- les totaux débit et crédit diffèrent.

## 4. Idempotence

Une même clé d’idempotence utilisée avec une requête strictement identique doit retourner le résultat déjà produit sans créer de nouvelles écritures.

Une même clé réutilisée avec un contenu différent doit être rejetée avec une erreur de conflit explicite et auditée.

La protection doit être imposée en base de données par une contrainte unique dans le périmètre métier approprié, et pas uniquement par un contrôle applicatif. Le schéma persiste également une empreinte déterministe `requestFingerprint` afin de distinguer une répétition exacte d’une collision de clé.

## 5. Compensation

Une transaction `POSTED` n’est jamais modifiée ni supprimée. Une correction crée une nouvelle transaction compensatoire qui inverse les effets comptables et référence la transaction d’origine.

La commande de compensation contient :

- l’identifiant de la transaction d’origine ;
- un code motif ;
- une clé d’idempotence ;
- un identifiant de corrélation.

Les opérations de compensation manuelle ou administrative doivent appliquer les règles RBAC/ABAC, la séparation des tâches et, selon le niveau de risque, une double validation.

## 6. Lecture et pagination

La lecture des écritures d’un compte accepte des bornes temporelles, un curseur et une limite. La pagination doit être stable : le curseur doit s’appuyer sur un ordre déterministe, idéalement une séquence comptable monotone plus l’identifiant de l’écriture.

Le schéma PostgreSQL maintient un index `(accountId, postedAt, id)` compatible avec une pagination keyset stable des écritures.

Les réponses ne doivent jamais contenir de secret, de PAN complet, de code OTP, de document KYC ou de donnée fournisseur non nécessaire.

## 7. Soldes

Le solde exposé par l’API est une projection du grand livre, pas une seconde source de vérité. Le modèle partagé expose actuellement :

- `available` ;
- `pending` ;
- `asOf`.

La persistance conserve en plus une position de projection via `projectionSequence`, `lastEntryPostedAt` et `lastEntryId`, afin de permettre la vérification et la reconstruction après incident.

## 8. Atomicité backend attendue

La publication d’une transaction doit être atomique avec :

1. validation des invariants ;
2. contrôle d’idempotence ;
3. verrouillage ou sérialisation appropriée ;
4. insertion de la transaction ;
5. insertion de toutes les écritures ;
6. mise à jour des projections de solde ;
7. insertion de l’événement dans l’outbox transactionnelle ;
8. écriture de l’audit technique ;
9. commit SQL unique.

Aucun événement externe ne doit être publié avant le commit de la transaction SQL.

## 9. Erreurs minimales

Le module backend doit mapper les erreurs métier vers les conventions API communes, notamment :

- écriture invalide ;
- transaction déséquilibrée ;
- conflit d’idempotence ;
- compte introuvable ou inactif ;
- devise incompatible ;
- transaction déjà compensée ;
- autorisation insuffisante ;
- indisponibilité temporaire ou conflit de concurrence.

Les messages publics restent génériques ; les détails techniques sont conservés dans les logs structurés avec l’identifiant de corrélation.

## 10. Critères d’acceptation

Le lot ledger n’est considéré terminé pour une mise en recette que si :

- les six routes du contrat sont implémentées et protégées ;
- les transactions déséquilibrées sont impossibles à publier ;
- les doublons idempotents ne créent aucune écriture supplémentaire ;
- les collisions d’idempotence sont détectées ;
- une compensation produit des écritures inverses et conserve le lien d’origine ;
- les soldes peuvent être reconstruits à partir des écritures ;
- les opérations concurrentes ne produisent ni perte ni double dépense ;
- l’outbox ne perd aucun événement après reprise ;
- les tests unitaires, intégration PostgreSQL et concurrence sont automatisés ;
- les métriques et alertes détectent les divergences de balance et les échecs de projection.

## 11. État actuel

Les modèles, invariants et routes de transport sont définis dans `packages/contracts/src/ledger.ts` et `packages/contracts/src/ledger-api.ts`.

Le schéma Prisma couvre les principales structures de persistance : comptes enrichis, transactions avec idempotence et corrélation, compensation, séquence des écritures, projection de solde et outbox transactionnelle. Les repositories PostgreSQL, l’orchestrateur de persistance et les lectures Prisma existent désormais pour le socle déjà construit.

L’API gateway expose actuellement les lectures de compte, solde et écritures, avec pagination keyset pour les écritures. Ces routes sont protégées par le guard de service interne et le comportement de ce guard est couvert par des tests Node dédiés.

Restent notamment à compléter avant production : les routes de publication, lecture d’une transaction et compensation, la migration Prisma générée et éprouvée en environnement PostgreSQL, le worker outbox, la réconciliation, les métriques/alertes, les tests d’intégration PostgreSQL réels et les scénarios de concurrence/reprise.
