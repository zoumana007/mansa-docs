# Migration vers l’isolation tenant du rapprochement

## 1. Objet

Ce document définit la procédure de migration sûre du module de rapprochement vers une isolation obligatoire par `organizationId`.

Il complète :

- `12-isolation-tenant-rapprochement.md` ;
- la spécification du rapprochement existante ;
- le plan technique `mansa-platform/docs/reconciliation-tenant-isolation.md`.

La migration ne doit jamais rendre visibles les lots ou lignes d’une organisation à une autre, ni casser l’idempotence d’import et de résolution.

## 2. Principe de déploiement

Le changement est réalisé en plusieurs étapes compatibles avec un déploiement progressif :

1. introduire la colonne d’organisation et les index sans supprimer immédiatement les contraintes historiques ;
2. attribuer une organisation explicite aux données existantes de développement/recette ;
3. rendre les écritures nouvelles obligatoirement scoppées ;
4. rendre les lectures et mutations obligatoirement scoppées ;
5. vérifier l’absence de lignes non attribuées ;
6. remplacer les anciennes contraintes d’unicité globales par des contraintes tenant-aware ;
7. rendre `organizationId` non nullable si une phase transitoire nullable a été utilisée ;
8. activer les tests inter-tenant bloquants dans la CI.

Aucune étape ne doit utiliser une organisation de production inventée. Les données historiques non attribuables doivent être isolées dans un tenant technique de migration uniquement dans les environnements non productifs, puis supprimées ou réattribuées explicitement.

## 3. Schéma cible

### ReconciliationBatch

Champs et contraintes minimales :

```text
organizationId      obligatoire
providerId          obligatoire
sourceFingerprint   obligatoire pour les imports idempotents
```

Unicité d’import :

```text
(organizationId, providerId, sourceFingerprint)
```

Index recommandés :

```text
(organizationId, status, createdAt)
(organizationId, providerId, status, createdAt)
(organizationId, periodStart, periodEnd)
```

### ReconciliationItem

`organizationId` est matérialisé sur la ligne en plus de la relation au lot afin de :

- rendre les requêtes de sécurité explicites ;
- faciliter les index ;
- simplifier les audits ;
- empêcher une future lecture par identifiant qui oublierait le lot parent.

Index recommandés :

```text
(organizationId, batchId, status)
(organizationId, batchId, mismatchReason)
(organizationId, internalReference)
(organizationId, providerReference)
(organizationId, createdAt, id)
```

## 4. Invariants obligatoires

À tout instant après la migration fonctionnelle :

- `ReconciliationItem.organizationId == ReconciliationBatch.organizationId` ;
- aucun import ne choisit son organisation depuis un fichier fournisseur ;
- aucun identifiant de lot ou d’item ne suffit à autoriser une lecture ;
- toutes les mutations combinent l’identifiant de ressource avec `organizationId` ;
- un curseur de pagination ne permet jamais de franchir une frontière de tenant ;
- l’idempotence d’import est locale à une organisation ;
- l’idempotence de résolution est locale à une organisation ;
- les compteurs du lot et les journaux d’audit ne changent jamais après une tentative inter-tenant refusée.

## 5. Migration des données existantes

Avant toute migration de production, exécuter un inventaire :

```text
nombre de lots
nombre d’items
sources distinctes
providers distincts
lots sans propriétaire métier connu
items orphelins éventuels
résolutions déjà effectuées
```

En développement et recette, les données de test historiques peuvent être attribuées à un tenant technique clairement nommé, par exemple `test-reconciliation-migration`, à condition que cette valeur ne soit jamais utilisée en production.

En production, aucune attribution automatique n’est autorisée sans source de vérité métier. Les lignes ambiguës restent bloquées hors exposition jusqu’à décision explicite.

## 6. Déploiement compatible

### Phase A — ajout structurel

Ajouter les colonnes et index organisationnels. Les anciennes routes continuent temporairement à fonctionner uniquement dans les environnements de développement si nécessaire.

### Phase B — double écriture contrôlée

Toutes les nouvelles créations écrivent `organizationId` sur le lot et chaque item. Une assertion transactionnelle empêche une divergence entre lot et items.

### Phase C — lectures scoppées

Toutes les méthodes de repository exigent `organizationId` :

```ts
getBatch(organizationId, batchId)
getItem(organizationId, itemId)
listBatches(organizationId, query)
listItems(organizationId, batchId, query)
```

Une ressource appartenant à un autre tenant est traitée comme inaccessible. La couche HTTP ne doit pas révéler son existence.

### Phase D — mutations scoppées

La résolution d’un item, la mise à jour des compteurs et l’écriture de l’audit sont exécutées dans la même transaction et avec la même organisation.

### Phase E — nettoyage

Après validation :

- supprimer l’ancienne unicité globale `(providerId, sourceFingerprint)` ;
- conserver uniquement l’unicité tenant-aware ;
- interdire les lignes sans organisation ;
- supprimer les chemins de compatibilité transitoires.

## 7. Rollback

Le rollback doit distinguer code et données.

Avant suppression d’une ancienne contrainte ou passage `NOT NULL`, conserver une sauvegarde vérifiée et un script de retour arrière testé en recette.

Un rollback ne doit jamais :

- supprimer `organizationId` de données déjà scoppées ;
- fusionner les imports de deux tenants partageant le même fournisseur et fingerprint ;
- rétablir une route capable de lire globalement les lots ;
- écraser les journaux d’audit.

Si l’application doit revenir à une version antérieure, l’exposition du module de rapprochement doit être désactivée plutôt que de réintroduire un accès non scoppé.

## 8. Tests de migration

La recette PostgreSQL doit couvrir au minimum :

1. migration d’une base vide ;
2. migration d’une base contenant des lots et items de test ;
3. même `providerId + sourceFingerprint` accepté dans deux organisations ;
4. doublon rejeté dans la même organisation ;
5. lecture d’un lot étranger impossible ;
6. lecture d’un item étranger impossible ;
7. résolution étrangère impossible ;
8. curseur d’un tenant réutilisé chez un autre sans fuite ;
9. compteurs inchangés après tentative interdite ;
10. audit inchangé après tentative interdite ;
11. cohérence `organizationId` lot/item vérifiée par requête SQL ;
12. rollback de recette vérifié avant toute production.

## 9. Requêtes de contrôle

Les contrôles de recette doivent rechercher :

```text
lots sans organizationId
items sans organizationId
items dont organizationId diffère du lot
sources dupliquées dans un même tenant
résolutions sans audit correspondant
```

Le résultat attendu pour les trois premières catégories est zéro.

## 10. Critères de passage

La migration est considérée prête lorsque :

- le schéma Prisma et les migrations SQL sont alignés ;
- tous les accès repository sont tenant-aware ;
- le service d’import reçoit l’organisation depuis un contexte interne autorisé ;
- les contrôleurs ne font pas confiance à une organisation fournie par un fournisseur ;
- les tests PostgreSQL inter-tenant passent ;
- les anciens chemins non scoppés sont supprimés ;
- la CI bloque toute régression ;
- la procédure de rollback a été testée en recette.

## 11. Hors périmètre immédiat

Cette migration ne remplace pas l’authentification workload définitive. Tant que celle-ci n’est pas disponible, l’API interne peut utiliser une identité de service transitoire, mais `organizationId` doit rester une donnée d’autorisation dérivée d’un contexte contrôlé et jamais une confiance aveugle dans un fichier ou une requête externe.

## 12. État d’implémentation

La phase structurelle est engagée dans `mansa-platform` :

- `ReconciliationBatch.organizationId` et `ReconciliationItem.organizationId` sont présents dans le schéma Prisma cible ;
- l’unicité d’import devient `(organizationId, providerId, sourceFingerprint)` ;
- l’idempotence de résolution devient locale au tenant via `(organizationId, resolutionIdempotencyKey)` ;
- les index de lecture et de pagination sont préfixés par `organizationId` ;
- une migration SQL dédiée refuse explicitement d’inventer une organisation lorsque des données historiques existent.

La migration automatique actuelle est donc volontairement sûre sur une base vide. Sur une base déjà peuplée, son échec est attendu tant qu’un backfill explicite n’a pas été réalisé selon le runbook. Cette protection empêche qu’une valeur de tenant arbitraire soit injectée silencieusement.

La prochaine tranche doit porter l’organisation jusque dans le repository, le service d’import, les routes internes et les tests PostgreSQL inter-tenant avant de considérer la phase structurelle comme activable en environnement partagé.
