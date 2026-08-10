# Isolation tenant du rapprochement financier

## 1. Objet

Cette spécification rend obligatoire l’isolation organisationnelle du moteur de rapprochement financier Mansa avant toute intégration partenaire réelle.

Le rapprochement traite des écritures, références de paiement, montants, statuts et informations opérationnelles sensibles. Aucun lot, item, filtre, résolution, compteur ou audit ne doit pouvoir franchir la frontière d’une organisation par simple connaissance d’un identifiant technique.

## 2. Identifiant de portée

La portée de référence est `organizationId`.

Elle doit être présente sur chaque `ReconciliationBatch`. Les items héritent obligatoirement de la portée du lot parent ; pour faciliter les contrôles et les requêtes de sécurité, l’implémentation matérialise également `organizationId` sur `ReconciliationItem`.

`organizationId` n’est jamais déduit d’une valeur fournie librement par le client final. Il provient à terme d’un contexte de service authentifié et autorisé. Dans l’étape transitoire actuelle, les routes internes protégées exigent explicitement cette portée afin d’empêcher tout appel repository non scoppé ; cette valeur explicite ne constitue pas encore une identité workload attestée.

## 3. Unicité des imports

L’idempotence d’import est scoppée par organisation :

```text
(organizationId, providerId, sourceFingerprint)
```

Deux organisations peuvent importer une source ayant le même fournisseur et la même empreinte sans collision entre tenants. Une même organisation ne peut créer qu’un lot pour cette clé.

## 4. Lectures

Toutes les lectures doivent exiger la portée d’organisation et l’appliquer dans la requête SQL/Prisma elle-même.

Sont concernés :

- liste des lots ;
- lecture d’un lot par identifiant ;
- liste des items d’un lot ;
- lecture d’un item par identifiant ;
- recherches par référence interne ;
- recherches par référence fournisseur ;
- exports et futurs rapports.

Le contrôle ne doit pas être réalisé uniquement après récupération d’une ligne non filtrée. Un identifiant appartenant à une autre organisation doit se comporter comme une ressource inexistante pour l’appelant : aucune fuite de présence, statut ou métadonnée.

## 5. Résolutions

Toute résolution `RESOLVED` ou `IGNORED` inclut la portée d’organisation dans la sélection de l’item et du lot parent.

Le rejeu d’idempotence est lui aussi isolé par organisation via :

```text
(organizationId, resolutionIdempotencyKey)
```

La transaction de résolution conserve dans la même unité atomique :

- la mutation de l’item ;
- le compteur du lot ;
- l’audit opérationnel ;
- l’`organizationId` dans les métadonnées d’audit.

Une tentative de résolution avec une organisation étrangère se comporte comme une ressource inexistante et ne doit créer ni mutation ni audit.

## 6. Pagination et curseurs

Les curseurs restent basés sur `(createdAt, id)` mais ne constituent jamais un mécanisme d’autorisation.

Une requête paginée applique toujours `organizationId` avant les conditions de curseur. Un curseur obtenu dans une organisation puis présenté dans une autre organisation ne doit permettre de récupérer aucune donnée étrangère.

## 7. Filtres de consultation

Les filtres du contrat partagé doivent être appliqués à l’intérieur de la portée organisationnelle :

### Lots

- `providerId` ;
- `status` ;
- `periodStartFrom` ;
- `periodEndTo`.

### Items

- `status` ;
- `mismatchReason` ;
- `internalReference` ;
- `providerReference` ;
- `createdFrom` ;
- `createdTo`.

Aucun filtre ne peut élargir la portée au-delà de l’organisation courante.

**État actuel :** la portée organisationnelle de base est implémentée sur lectures, listes, import et résolution. L’application des filtres du contrat partagé constitue le prochain sous-lot avant de considérer cette tranche entièrement terminée.

## 8. Frontière API interne

À court terme, tant que l’identité workload attestée n’est pas encore déployée, les méthodes internes rendent la portée explicite et obligatoire afin d’éviter tout appel repository non scoppé.

La cible finale est :

```text
workload authentifié
→ organisation(s) autorisée(s)
→ contexte de requête interne
→ repository toujours scoppé
→ réponse DTO sérialisée
```

Il est interdit de considérer un simple header ou paramètre arbitraire comme preuve d’appartenance à une organisation en production.

## 9. Schéma PostgreSQL

`ReconciliationBatch` comporte désormais :

```text
organizationId String
```

avec :

```text
UNIQUE (organizationId, providerId, sourceFingerprint)
INDEX  (organizationId, status, createdAt)
INDEX  (organizationId, providerId, status, createdAt)
INDEX  (organizationId, periodStart, periodEnd)
```

`ReconciliationItem.organizationId` est matérialisé avec notamment :

```text
UNIQUE (organizationId, resolutionIdempotencyKey)
INDEX  (organizationId, batchId, status)
INDEX  (organizationId, batchId, mismatchReason)
INDEX  (organizationId, internalReference)
INDEX  (organizationId, providerReference)
INDEX  (organizationId, createdAt, id)
```

La migration versionnée refuse volontairement de rendre `organizationId` obligatoire sur une base déjà peuplée sans backfill explicite. Le runbook de migration décrit la procédure contrôlée ; aucune organisation réelle n’est inventée silencieusement.

## 10. Tests obligatoires

La validation PostgreSQL doit démontrer au minimum :

1. deux organisations peuvent importer la même paire fournisseur/empreinte et obtiennent deux lots distincts ;
2. un rejeu dans la même organisation réutilise le même lot ;
3. `getBatch` ne retourne pas le lot d’une autre organisation ;
4. `getItem` ne retourne pas l’item d’une autre organisation ;
5. `listBatches` ne mélange jamais les organisations ;
6. `listItems` ne mélange jamais les organisations ;
7. les curseurs ne permettent pas de franchir la portée ;
8. une résolution inter-tenant est refusée comme ressource inexistante ;
9. les compteurs du lot étranger restent inchangés ;
10. aucun audit n’est créé pour une tentative inter-tenant ;
11. l’idempotence de résolution ne provoque aucune collision entre organisations ;
12. les filtres sont cumulables sans élargir la portée.

Les tests runtime et PostgreSQL couvrent désormais les points 1 à 10 dans le chemin principal de rapprochement. Les scénarios de filtres cumulables et la collision de clés de résolution identiques entre deux organisations restent à compléter dans la tranche suivante.

## 11. Sérialisation

L’isolation tenant ne dispense pas de sérialiser les réponses selon les DTO publics partagés.

Les champs internes tels que :

- empreintes techniques de source ;
- empreintes de ligne brute ;
- métadonnées d’adaptateur ;
- clés d’idempotence de résolution ;
- détails internes de persistance ;

doivent rester absents des réponses HTTP sauf contrat explicite.

Cette sérialisation stricte n’est pas encore considérée comme terminée.

## 12. État d’implémentation

Implémenté :

- colonnes et index organisationnels dans Prisma ;
- migration PostgreSQL versionnée avec garde de backfill ;
- propagation de `organizationId` lors des imports ;
- idempotence d’import par organisation ;
- lectures et listes scoppées directement dans Prisma ;
- résolution et rejeu d’idempotence scoppés ;
- métadonnée d’organisation dans l’audit de résolution ;
- routes internes exigeant une portée explicite ;
- tests contrôleur et PostgreSQL des principaux scénarios inter-tenant.

Restant avant clôture de cette tranche :

- filtres de consultation du contrat partagé ;
- test explicite d’une même clé de résolution utilisée indépendamment dans deux organisations ;
- sérialisation stricte des réponses HTTP vers les DTO publics ;
- remplacement de la portée explicite transitoire par une identité workload attestée avant production réelle.

## 13. Suite

Le prochain lot doit implémenter les filtres de consultation dans le repository et le contrôleur sans jamais élargir la portée tenant. Il doit ensuite sérialiser strictement les réponses HTTP vers `ReconciliationBatchSummary`, `ReconciliationItem` et `PageResponse`. L’identité workload et l’observabilité viennent après ces deux étapes, avant tout adaptateur partenaire réel.
