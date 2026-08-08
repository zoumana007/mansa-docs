# Publication transactionnelle du ledger

## 1. Objet

Ce document décrit la première implémentation exécutable de publication atomique des transactions comptables dans `mansa-platform/apps/api-gateway`.

Le point d’entrée interne est :

```text
POST /v1/internal/ledger/transactions
```

La route reste protégée par le guard des services internes et ne doit pas être exposée directement aux applications clientes.

## 2. Chaîne de traitement

Une commande acceptée suit l’ordre suivant :

1. validation HTTP déterministe de l’enveloppe et des écritures ;
2. calcul d’une empreinte SHA-256 canonique de la requête ;
3. ouverture d’une transaction Prisma/PostgreSQL ;
4. recherche de la clé d’idempotence ;
5. retour du résultat existant si la même clé correspond à la même empreinte ;
6. rejet si la clé correspond à une requête différente ;
7. chargement de tous les comptes utilisés ;
8. validation de l’existence, de la devise et du pays des comptes ;
9. création de la transaction comptable au statut `POSTED` ;
10. insertion de toutes les écritures ;
11. mise à jour atomique des projections de solde ;
12. création d’un événement outbox `ledger.transaction.posted.v1` ;
13. validation de la transaction SQL.

Aucune écriture, projection ou entrée d’outbox ne doit être confirmée isolément si une étape du bloc transactionnel échoue.

## 3. Idempotence

`LedgerTransaction.idempotencyKey` est unique.

Le backend conserve aussi `requestFingerprint`. Une répétition strictement identique retourne la transaction déjà publiée avec :

```json
{
  "replayed": true
}
```

La réutilisation d’une clé avec un autre contenu doit répondre par un conflit et ne produire aucune nouvelle écriture.

L’empreinte inclut au minimum :

- référence métier ;
- type de transaction ;
- écritures dans leur ordre ;
- montants en unités mineures ;
- devise ;
- clé d’idempotence ;
- identifiant de corrélation ;
- pays ;
- date métier ;
- métadonnées triées par clé.

## 4. Validation des comptes

Avant publication, chaque compte référencé doit :

- exister ;
- utiliser la même devise que l’écriture ;
- appartenir au pays de la commande.

Les contrôles produit plus avancés restent à ajouter : comptes fermés ou gelés, restrictions réglementaires, autorisations par domaine métier et règles de contrepartie.

## 5. Projection de solde

La projection `LedgerBalanceProjection.availableMinor` est mise à jour suivant le sens naturel du type de compte :

- `ASSET` et `EXPENSE` : le débit augmente le solde naturel ;
- `LIABILITY`, `EQUITY` et `REVENUE` : le crédit augmente le solde naturel.

Le sens opposé diminue la projection.

La projection reste reconstructible à partir des écritures et n’est jamais la source de vérité financière.

Chaque écriture met également à jour :

- `lastEntryPostedAt` ;
- `lastEntryId` ;
- `projectionSequence`.

## 6. Outbox

La publication crée dans la même transaction SQL un événement :

```text
ledger.transaction.posted.v1
```

Le payload minimal contient les identifiants et informations de corrélation nécessaires aux workers sans inclure de secret ni de donnée de paiement sensible.

La livraison asynchrone de l’outbox reste séparée de la publication comptable. Un worker devra récupérer les événements `PENDING`, les publier de manière idempotente, puis passer leur statut à `PUBLISHED` ou gérer les tentatives et erreurs.

## 7. Erreurs attendues

Les erreurs fonctionnelles minimales sont :

- `400` : commande HTTP ou partie double invalide ;
- `409` : collision d’idempotence avec un contenu différent ;
- `422` : compte inexistant ou incohérence compte/devise/pays ;
- `401/403` : appel interne non autorisé selon le guard en vigueur.

Les erreurs internes ne doivent pas exposer les requêtes SQL, secrets, tokens ni détails de configuration.

## 8. Garanties acquises par ce lot

Le socle couvre désormais :

- validation HTTP des commandes ;
- route interne de publication ;
- transaction Prisma atomique ;
- persistance de la transaction et des écritures ;
- mise à jour des projections ;
- création transactionnelle de l’outbox ;
- répétition idempotente identique ;
- détection d’une collision d’idempotence différente ;
- contrôle compte/devise/pays.

## 9. Travaux suivants

Les lots suivants doivent compléter ce socle avec :

1. gestion explicite des collisions d’idempotence concurrentes au niveau PostgreSQL ;
2. verrouillage et stratégie de concurrence documentée sur les comptes/projections ;
3. compensation complète d’une transaction publiée ;
4. worker outbox avec reprise et backoff ;
5. tests d’intégration sur PostgreSQL réel ;
6. tests de concurrence et de double soumission ;
7. reconstruction et réconciliation automatique des projections ;
8. métriques, alertes et runbook d’intégrité du ledger.

## 10. Fichiers de référence

Dans `mansa-platform` :

- `apps/api-gateway/src/ledger-write.validation.ts` ;
- `apps/api-gateway/src/ledger-write.service.ts` ;
- `apps/api-gateway/src/ledger.controller.ts` ;
- `apps/api-gateway/src/ledger.module.ts` ;
- `apps/api-gateway/prisma/schema.prisma`.

Ce document complète `09-grand-livre-et-integrite-financiere.md` et `10-contrat-api-ledger.md` sans les remplacer.
