# Validation PostgreSQL du rapprochement financier

## 1. Objet

Ce document complète `10-moteur-rapprochement-financier.md` en décrivant la tranche de validation réelle de la persistance PostgreSQL du moteur de rapprochement Mansa.

L'objectif est de vérifier les propriétés qui ne peuvent pas être démontrées uniquement par des tests unitaires du moteur pur ou par la validation statique du schéma Prisma : transactionnalité, normalisation persistée, idempotence réelle, comportement sous concurrence et bornes de lecture.

## 2. Périmètre testé

Le test d'intégration de référence se trouve dans :

`mansa-platform/apps/api-gateway/test/reconciliation-postgres.test.mjs`.

Il est volontairement opt-in pour éviter qu'un environnement de développement sans PostgreSQL échoue lors des tests ordinaires. Son exécution exige :

```text
RUN_POSTGRES_TESTS=1
DATABASE_URL=postgresql://...
```

Le script dédié du package API est :

```bash
pnpm --filter @mansa/api-gateway test:postgres
```

## 3. Environnement CI

La CI principale de `mansa-platform` démarre un service PostgreSQL éphémère, génère le client Prisma, applique les migrations versionnées puis exécute les tests classiques et le test d'intégration PostgreSQL.

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

Cette protection est nécessaire avant de brancher des workers ou plusieurs réplicas API sur les imports de fichiers de règlement.

## 7. Lectures bornées

Les méthodes de lecture doivent rester bornées même lorsqu'un appelant demande une limite excessive.

La validation PostgreSQL confirme actuellement :

- `listBatches` : maximum 100 lots ;
- `listItems` : maximum 500 items par lot.

Ces plafonds protègent la base et l'API interne contre les lectures accidentellement non bornées. Ils ne remplacent pas la pagination par curseur prévue dans le contrat partagé.

## 8. Nettoyage des données de test

Chaque exécution utilise des identifiants fournisseur et empreintes uniques. Les lots et items créés sont supprimés en fin de test avant déconnexion Prisma.

Le test ne dépend d'aucune donnée de production et ne doit jamais être pointé vers une base réelle. Les environnements de recette ou de production doivent utiliser des procédures de validation séparées et explicitement autorisées.

## 9. Cohérence avec le contrat fonctionnel

Cette tranche couvre les critères de recette suivants du moteur de rapprochement :

- persistance réelle des lots et items ;
- réutilisation d'une source déjà importée ;
- unicité fournisseur/empreinte ;
- concurrence d'import ;
- normalisation aux frontières ;
- compteurs matérialisés ;
- lectures bornées.

Elle ne clôt pas le module complet.

## 10. Étapes suivantes

Les prochaines tranches cohérentes sont :

1. tests de contrat HTTP des routes internes de consultation avec identité de service valide/invalide ;
2. pagination par curseur alignée sur `@mansa/contracts/reconciliation-api` ;
3. résolution manuelle atomique des écarts avec clé d'idempotence ;
4. `OperationalAuditLog` dans la même transaction que la résolution ;
5. isolation tenant/organisation complète ;
6. métriques et alertes de rapprochement ;
7. premiers adaptateurs partenaires réels, sans secret versionné.

Aucun endpoint de résolution ne doit être exposé avant que l'autorisation, l'idempotence et l'audit transactionnel soient couverts par des tests.
