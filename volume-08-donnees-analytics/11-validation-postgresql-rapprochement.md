# Validation PostgreSQL du rapprochement financier

## 1. Objet

Ce document complète `10-moteur-rapprochement-financier.md` en décrivant la validation réelle de la persistance PostgreSQL et des principales frontières API du moteur de rapprochement Mansa.

L'objectif est de vérifier les propriétés qui ne peuvent pas être démontrées uniquement par des tests unitaires du moteur pur ou par la validation statique du schéma Prisma : transactionnalité, normalisation persistée, idempotence réelle, comportement sous concurrence, pagination stable, résolution manuelle atomique, audit opérationnel, isolation organisationnelle, filtres et sérialisation stricte des DTO HTTP.

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

## 5. Idempotence et concurrence

La clé fonctionnelle d'idempotence de l'import est désormais scoppée par organisation :

```text
(organizationId, providerId, sourceFingerprint)
```

Un second import portant exactement la même clé doit retourner le lot déjà persisté avec `reused = true` au lieu de créer un doublon.

La protection sous concurrence repose sur la contrainte unique PostgreSQL et la récupération applicative de la collision `P2002`, afin de relire le lot créé par le concurrent gagnant.

Les tests vérifient qu'une même source peut exister dans deux organisations différentes tout en restant idempotente à l'intérieur de chaque tenant.

## 6. Pagination par curseur

Les lectures de lots et d'items utilisent une pagination keyset stable avec enveloppe :

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

## 7. Isolation organisationnelle

`organizationId` est matérialisé sur les lots et les items puis imposé dans les requêtes Prisma de lecture, pagination, import et résolution.

La recette PostgreSQL valide notamment :

- absence de lecture d'un lot d'un autre tenant ;
- absence de lecture d'un item d'un autre tenant ;
- listes limitées au tenant courant ;
- pagination ne traversant pas les organisations ;
- coexistence de la même source fournisseur dans deux organisations ;
- refus d'une résolution inter-tenant ;
- absence d'audit ou de compteur modifié lors d'une tentative inter-tenant.

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

L'audit conserve : corrélation, acteur, type d'acteur, action, ressource, motif, note de résolution, lot parent, motif d'écart initial et portée organisationnelle.

Un échec d'audit doit annuler la résolution au lieu de produire une mutation non traçable.

## 10. Filtres de consultation

Les routes de lots appliquent dans Prisma :

- `providerId` ;
- `status` ;
- `periodStartFrom` ;
- `periodEndTo`.

Les routes d'items appliquent :

- `providerId` via le lot parent ;
- `status` ;
- `mismatchReason` ;
- `internalReference` ;
- `providerReference` ;
- `createdFrom` ;
- `createdTo`.

Les enums sont validées contre les listes partagées du package de contrats. Les dates invalides sont rejetées en `400`. Les filtres sont combinés avec la portée organisationnelle et la pagination keyset dans les requêtes de persistance, et non appliqués après chargement.

## 11. Sérialisation HTTP stricte

Le contrôleur ne renvoie plus directement les objets Prisma.

Les lots sont sérialisés vers `ReconciliationBatchSummary` et les items vers `ReconciliationItem` grâce à `reconciliation.presenter.ts`.

La validation de transport couvre :

- dates converties en chaînes ISO 8601 ;
- montants `bigint` convertis seulement lorsqu'ils sont des entiers sûrs ;
- absence de `organizationId` dans le DTO public ;
- absence des empreintes de source et de ligne ;
- absence des métadonnées Prisma internes ;
- absence des clés/corrélations d'idempotence de résolution dans la réponse ;
- même sérialisation stricte après une résolution.

Le détail fonctionnel de cette tranche est documenté dans `13-filtres-et-contrats-http-rapprochement.md`.

## 12. Validation du contrôleur interne

`reconciliation-controller.test.mjs` couvre désormais :

- portée organisationnelle obligatoire ;
- limites par défaut et bornes maximales ;
- transmission et rejet des curseurs invalides ;
- filtres de lots et d'items ;
- rejet des enums et dates invalides ;
- `404` sur lot ou item absent ;
- validation de `RESOLVED | IGNORED` ;
- transformation des erreurs métier attendues en erreurs HTTP ;
- transmission intégrale des champs nécessaires à l'idempotence et à l'audit ;
- sérialisation des DTO et absence de fuite des champs internes.

Toutes les routes internes restent protégées par `InternalServiceGuard`.

## 13. Nettoyage des données de test

Chaque exécution PostgreSQL utilise des identifiants fournisseur et empreintes uniques. Les audits, items et lots créés sont supprimés en fin de test avant déconnexion Prisma.

Le test ne dépend d'aucune donnée de production et ne doit jamais être pointé vers une base réelle. Les environnements de recette ou de production doivent utiliser des procédures de validation séparées et explicitement autorisées.

## 14. État et étapes suivantes

Cette tranche couvre désormais :

- persistance réelle des lots et items ;
- idempotence séquentielle et concurrente ;
- normalisation aux frontières ;
- compteurs matérialisés ;
- pagination par curseur ;
- isolation organisationnelle ;
- résolution manuelle idempotente ;
- audit atomique ;
- filtres de consultation ;
- sérialisation stricte des DTO HTTP ;
- validation du contrôleur interne.

Elle ne clôt pas le module complet.

Les prochaines tranches cohérentes sont :

1. identité workload attestée et dérivation de la portée autorisée ;
2. métriques et alertes de rapprochement ;
3. premiers adaptateurs partenaires réels, sans secret versionné ;
4. tests de charge et de concurrence élargis.

Aucune exposition à des partenaires externes ne doit précéder la validation de l'identité workload.
