# Validation PostgreSQL du rapprochement financier

## 1. Objet

Ce document complète `10-moteur-rapprochement-financier.md` en décrivant la validation réelle de la persistance PostgreSQL et des principales frontières API du moteur de rapprochement Mansa.

L'objectif est de vérifier les propriétés qui ne peuvent pas être démontrées uniquement par des tests unitaires du moteur pur ou par la validation statique du schéma Prisma : transactionnalité, normalisation persistée, idempotence réelle, comportement sous concurrence, pagination stable, résolution manuelle atomique et audit opérationnel.

## 2. Périmètre testé

Les tests de référence se trouvent dans :

- `mansa-platform/apps/api-gateway/test/reconciliation-postgres.test.mjs` ;
- `mansa-platform/apps/api-gateway/test/reconciliation-controller.test.mjs`.

Le test PostgreSQL est volontairement opt-in pour éviter qu'un environnement de développement sans PostgreSQL échoue lors des tests ordinaires. Son exécution exige :

```text
RUN_POSTGRES_TESTS=1
DATABASE_URL=postgresql://...
```

Le script dédié du package API est :

```bash
pnpm --filter @mansa/api-gateway test:postgres
```

Les tests de contrôleur font partie de la suite standard `pnpm test` du package API.

## 3. Environnement CI

La CI principale de `mansa-platform` démarre un service PostgreSQL éphémère, génère le client Prisma, applique les migrations versionnées puis exécute les tests classiques et les tests d'intégration PostgreSQL.

Le compte et le mot de passe utilisés dans la CI sont uniquement des valeurs de test locales au job GitHub Actions. Aucun secret de production n'est stocké dans le dépôt.

Ordre de validation :

1. installation des dépendances ;
2. validation du registre produit ;
3. validation du schéma Prisma ;
4. génération du client Prisma ;
5. `prisma migrate deploy` sur la base éphémère ;
6. format, lint et typecheck ;
7. tests unitaires et de contrat ;
8. tests PostgreSQL du rapprochement ;
9. build complet.

## 4. Persistance atomique

Le test vérifie qu'un import contenant des éléments rapprochés et en écart :

- crée un seul `ReconciliationBatch` ;
- crée exactement les `ReconciliationItem` attendus ;
- normalise la devise en majuscules ;
- normalise les références et statuts aux frontières du repository ;
- matérialise les compteurs `totalItems`, `matchedItems` et `mismatchedItems` ;
- termine le lot dans `COMPLETED` ou `COMPLETED_WITH_MISMATCHES` ;
- renseigne `completedAt` ;
- conserve la totalité dans une même transaction applicative.

Un échec de création des items ou de mise à jour finale doit provoquer le rollback de la transaction au lieu de laisser un lot partiellement finalisé.

## 5. Idempotence séquentielle

La clé fonctionnelle d'idempotence de l'import est :

```text
(providerId, sourceFingerprint)
```

Un second import portant exactement la même paire doit retourner le lot déjà persisté avec `reused = true` au lieu de créer un doublon.

Les compteurs retournés sont ceux du lot existant, même si le second appel présente une liste d'items différente. La source déjà traitée reste la source de vérité de cette exécution.

## 6. Idempotence concurrente

La simple stratégie « rechercher puis créer » n'est pas suffisante sous concurrence : plusieurs workers peuvent observer simultanément l'absence du lot avant de tenter sa création.

La protection de référence repose donc sur deux niveaux :

1. contrainte unique PostgreSQL sur `(providerId, sourceFingerprint)` ;
2. récupération applicative de la collision `P2002` afin de relire le lot créé par le concurrent gagnant et retourner `reused = true`.

Le test lance plusieurs imports simultanés de la même source et vérifie :

- un seul identifiant de lot final ;
- une seule création effective ;
- les autres appels signalés comme réutilisation ;
- une seule ligne de lot pour la paire fournisseur/empreinte ;
- aucun doublon d'item produit par les appels perdants.

## 7. Pagination par curseur

Les lectures de lots et d'items utilisent désormais une pagination keyset stable avec enveloppe :

```text
{
  data: [...],
  page: {
    hasNextPage: boolean,
    nextCursor?: string
  }
}
```

Les curseurs encodent le couple `(createdAt, id)` afin de départager les lignes partageant le même timestamp.

Ordre de référence :

- lots : `createdAt DESC, id DESC` ;
- items : `createdAt ASC, id ASC`.

Limites maximales :

- `listBatches` : 100 ;
- `listItems` : 500.

Un curseur illisible ou mal formé est rejeté au niveau API avec une erreur `400` plutôt que d'être interprété silencieusement.

## 8. Résolution manuelle atomique

Un écart `MISMATCHED` ou `PARTIALLY_MATCHED` peut être clôturé en `RESOLVED` ou `IGNORED` uniquement via une commande explicite contenant au minimum :

```text
resolutionNote
reasonCode
idempotencyKey
correlationId
actorId
actorType
```

La même clé d'idempotence rejouée avec la même ressource et le même résultat retourne la résolution existante sans incrémenter de nouveau les compteurs.

Une clé déjà utilisée pour une autre ressource ou un autre résultat est rejetée.

## 9. Audit opérationnel transactionnel

La mutation de l'item, l'incrément de `resolvedItems` ou `ignoredItems` sur le lot et la création de `OperationalAuditLog` sont exécutés dans la même transaction Prisma/PostgreSQL.

L'audit conserve :

- corrélation ;
- acteur ;
- type d'acteur ;
- action `RECONCILIATION_ITEM_RESOLVED` ou `RECONCILIATION_ITEM_IGNORED` ;
- ressource ;
- motif ;
- note de résolution ;
- lot parent ;
- motif d'écart initial.

Un échec d'audit doit annuler la résolution au lieu de produire une mutation non traçable.

## 10. Validation du contrôleur interne

`reconciliation-controller.test.mjs` couvre les frontières de transport déjà implémentées :

- limites par défaut ;
- bornes maximales ;
- transmission du curseur ;
- rejet des limites invalides ;
- rejet des curseurs invalides ;
- `404` sur lot ou item absent ;
- validation de `RESOLVED | IGNORED` ;
- transformation des erreurs métier attendues en erreurs HTTP ;
- transmission intégrale des champs nécessaires à l'idempotence et à l'audit.

Toutes les routes internes restent protégées par `InternalServiceGuard`.

## 11. Nettoyage des données de test

Chaque exécution PostgreSQL utilise des identifiants fournisseur et empreintes uniques. Les audits, items et lots créés sont supprimés en fin de test avant déconnexion Prisma.

Le test ne dépend d'aucune donnée de production et ne doit jamais être pointé vers une base réelle. Les environnements de recette ou de production doivent utiliser des procédures de validation séparées et explicitement autorisées.

## 12. Cohérence avec le contrat fonctionnel

Cette tranche couvre désormais :

- persistance réelle des lots et items ;
- réutilisation d'une source déjà importée ;
- unicité fournisseur/empreinte ;
- concurrence d'import ;
- normalisation aux frontières ;
- compteurs matérialisés ;
- pagination par curseur ;
- lectures bornées ;
- résolution manuelle idempotente ;
- audit atomique ;
- validation du contrôleur interne.

Elle ne clôt pas le module complet.

## 13. Étapes suivantes

Les prochaines tranches cohérentes sont :

1. isolation tenant/organisation complète des lots, items, recherches et résolutions ;
2. filtres de consultation prévus par `@mansa/contracts/reconciliation-api` ;
3. sérialisation stricte des réponses HTTP selon les DTO partagés ;
4. identité workload attestée pour les routes internes ;
5. métriques et alertes de rapprochement ;
6. premiers adaptateurs partenaires réels, sans secret versionné.

Aucune exposition à des partenaires externes ne doit précéder l'isolation tenant et la validation d'identité workload.
