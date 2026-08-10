# Isolation tenant du rapprochement financier

## 1. Objet

Cette spécification rend obligatoire l’isolation organisationnelle du moteur de rapprochement financier Mansa avant toute intégration partenaire réelle.

Le rapprochement traite des écritures, références de paiement, montants, statuts et informations opérationnelles sensibles. Aucun lot, item, filtre, résolution, compteur ou audit ne doit pouvoir franchir la frontière d’une organisation par simple connaissance d’un identifiant technique.

## 2. Identifiant de portée

La portée de référence est `organizationId`.

Elle doit être présente sur chaque `ReconciliationBatch`. Les items héritent obligatoirement de la portée du lot parent ; pour faciliter les contrôles et les requêtes de sécurité, l’implémentation peut également matérialiser `organizationId` sur `ReconciliationItem`, à condition qu’une contrainte applicative et les tests garantissent l’égalité avec le lot parent.

`organizationId` n’est jamais déduit d’une valeur fournie librement par le client final. Il provient d’un contexte de service authentifié et autorisé.

## 3. Unicité des imports

L’idempotence d’import ne doit plus être globale par fournisseur.

La clé fonctionnelle devient :

```text
(organizationId, providerId, sourceFingerprint)
```

Deux organisations peuvent importer une source ayant le même fournisseur et la même empreinte sans collision entre tenants.

Une même organisation ne peut créer qu’un lot pour cette clé.

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

Le contrôle ne doit pas être réalisé uniquement après récupération d’une ligne non filtrée.

Un identifiant appartenant à une autre organisation doit se comporter comme une ressource inexistante pour l’appelant : aucune fuite de présence, statut ou métadonnée.

## 5. Résolutions

Toute résolution `RESOLVED` ou `IGNORED` doit inclure la portée d’organisation dans la sélection de l’item et du lot parent.

Le rejeu d’idempotence est lui aussi isolé par organisation. Une clé utilisée par une organisation ne doit ni bloquer ni révéler une résolution appartenant à une autre organisation.

La transaction de résolution conserve dans la même unité atomique :

- la mutation de l’item ;
- le compteur du lot ;
- l’audit opérationnel ;
- l’`organizationId` dans les métadonnées ou dans un champ dédié lorsque le modèle d’audit l’introduira.

## 6. Pagination et curseurs

Les curseurs restent basés sur `(createdAt, id)` mais ne constituent jamais un mécanisme d’autorisation.

Une requête paginée applique toujours `organizationId` avant les conditions de curseur. Un curseur obtenu dans une organisation puis présenté dans une autre organisation ne doit permettre de récupérer aucune donnée étrangère.

## 7. Filtres de consultation

Les filtres du contrat partagé sont appliqués à l’intérieur de la portée organisationnelle :

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

## 8. Frontière API interne

À court terme, tant que l’identité workload attestée n’est pas encore déployée, les méthodes internes doivent rendre la portée explicite et obligatoire afin d’éviter tout appel repository non scoppé.

La cible finale est :

```text
workload authentifié
→ organisation(s) autorisée(s)
→ contexte de requête interne
→ repository toujours scoppé
→ réponse DTO sérialisée
```

Il est interdit de considérer un simple header arbitraire comme preuve d’appartenance à une organisation en production.

## 9. Schéma PostgreSQL cible

`ReconciliationBatch` doit comporter :

```text
organizationId String
```

avec au minimum :

```text
UNIQUE (organizationId, providerId, sourceFingerprint)
INDEX  (organizationId, status, createdAt)
INDEX  (organizationId, providerId, createdAt)
INDEX  (organizationId, periodStart, periodEnd)
```

Si `ReconciliationItem.organizationId` est matérialisé :

```text
INDEX (organizationId, batchId, status)
INDEX (organizationId, mismatchReason, createdAt)
INDEX (organizationId, internalReference)
INDEX (organizationId, providerReference)
```

La migration ne doit pas inventer une organisation réelle pour les anciennes données. Les environnements contenant déjà des lignes doivent utiliser une valeur de migration explicitement documentée et réservée au développement/test, ou un backfill contrôlé avant de rendre la colonne `NOT NULL`.

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

## 11. Sérialisation

L’isolation tenant ne dispense pas de sérialiser les réponses selon les DTO publics partagés.

Les champs internes tels que :

- empreintes techniques de source ;
- empreintes de ligne brute ;
- métadonnées d’adaptateur ;
- clés d’idempotence de résolution ;
- détails internes de persistance ;

doivent rester absents des réponses HTTP sauf contrat explicite.

## 12. Critères d’acceptation

Cette tranche est terminée lorsque :

- le schéma et la migration sont versionnés ;
- tous les accès repository exigent `organizationId` ;
- l’import est idempotent dans la portée organisationnelle ;
- toutes les routes de consultation et résolution sont scoppées ;
- les tests PostgreSQL et contrôleur couvrent les tentatives inter-tenant ;
- les filtres du contrat partagé fonctionnent dans cette portée ;
- la documentation et le README technique décrivent l’état réel sans prétendre qu’une identité workload non implémentée est déjà disponible.

## 13. Suite

Après validation de cette isolation, la tranche suivante doit sérialiser strictement les réponses HTTP vers `ReconciliationBatchSummary`, `ReconciliationItem` et `PageResponse`, puis renforcer l’identité workload et l’observabilité avant tout adaptateur partenaire réel.
