# Contrat API interne — Accès et mobilité

## 1. Objet

Ce document aligne le contrat partagé `@mansa/contracts/access-mobility-api` avec l’API Gateway réellement exposée par Mansa. Le moteur d’accès reste un service interne protégé : les applications clientes, portails, terminaux et adaptateurs partenaires ne doivent pas contourner les contrôles de l’API Gateway ni appeler directement la persistance.

Le namespace de référence est :

```text
/v1/internal/access
```

Aucune route de gestion du moteur d’accès ne doit être publiée sous `/v1/access` sans décision d’architecture explicite, authentification utilisateur adaptée, autorisations métier et tests dédiés.

## 2. Protection commune

Les routes NestJS actuelles sont placées derrière `InternalServiceGuard`.

Cela constitue une protection transitoire de service à service. Avant production, l’identité de workload doit être attestée et les permissions doivent être réduites au strict nécessaire par appelant.

Les règles suivantes restent obligatoires :

- aucune clé ou identité de service réelle dans Git ;
- corrélation des appels sensibles ;
- isolation par `organizationId` ;
- validation des entrées côté serveur ;
- absence de confiance implicite dans les lecteurs terrain ;
- audit des mutations administratives et décisions sensibles.

## 3. Routes contractuelles

Le contrat partagé expose les routes suivantes :

```text
POST /v1/internal/access/credentials
GET  /v1/internal/access/credentials/:credentialId
GET  /v1/internal/access/credentials

POST /v1/internal/access/entitlements
GET  /v1/internal/access/entitlements/:entitlementId
GET  /v1/internal/access/entitlements

POST /v1/internal/access/evaluate
POST /v1/internal/access/usages

GET  /v1/internal/access/locations/:locationId/availability
PUT  /v1/internal/access/locations/:locationId/availability

GET  /v1/internal/access/terminals/:terminalId/profile
GET  /v1/internal/access/terminals/:terminalId/display-state
POST /v1/internal/access/terminals/:terminalId/cash-validations
```

Le contrat définit le périmètre cible. Toutes les routes listées ne sont pas encore implémentées dans le contrôleur courant.

## 4. Routes actuellement exécutables

La tranche engagée dans `mansa-platform/apps/api-gateway/src/access/access.controller.ts` couvre actuellement :

```text
GET  /v1/internal/access/credentials/:credentialId
GET  /v1/internal/access/credentials
GET  /v1/internal/access/entitlements/:entitlementId
GET  /v1/internal/access/entitlements
POST /v1/internal/access/evaluate
```

Les consultations exigent `organizationId` et appliquent l’isolation tenant dans le repository Prisma. Les listes sont bornées à 100 éléments dans la surface HTTP actuelle.

Les créations, l’enregistrement d’usage, la gestion de disponibilité, les profils terminaux et les validations espèces restent contractuels mais doivent encore être exposés de façon sûre dans l’API Gateway.

## 5. Consultation d’un credential

Requête :

```text
GET /v1/internal/access/credentials/:credentialId?organizationId=...
```

Le repository recherche simultanément l’identifiant et l’organisation. Un identifiant existant dans une autre organisation ne doit pas être révélé.

Réponses attendues :

- `200` avec le credential si l’organisation correspond ;
- `400` si `organizationId` est absent ou vide ;
- `404` si aucun credential n’est visible dans ce tenant.

## 6. Liste des credentials

Requête :

```text
GET /v1/internal/access/credentials?organizationId=...&subjectId=...&status=...&credentialType=...&limit=...
```

Filtres supportés par la tranche actuelle :

- `organizationId` obligatoire ;
- `subjectId` facultatif ;
- `status` facultatif ;
- `credentialType` facultatif ;
- `limit` facultatif, compris entre 1 et 100.

Le champ `cursor` existe dans le contrat partagé mais n’est pas encore branché sur le contrôleur de lecture. L’implémentation ne doit donc pas prétendre fournir une pagination complète tant que cette partie n’est pas livrée et testée.

## 7. Consultation des entitlements

Les mêmes principes s’appliquent aux droits :

```text
GET /v1/internal/access/entitlements/:entitlementId?organizationId=...
GET /v1/internal/access/entitlements?organizationId=...&subjectId=...&useCase=...&status=...&limit=...
```

L’organisation est toujours obligatoire et fait partie du filtre Prisma.

Un droit d’une autre organisation doit être indistinguable d’un droit absent pour l’appelant.

## 8. Évaluation d’accès

Requête :

```text
POST /v1/internal/access/evaluate
```

Champs minimaux actuellement exigés :

```text
requestId
organizationId
useCase
credentialType
credentialReference
locationId
occurredAt
correlationId
```

Le contrôleur transmet ensuite la requête au service applicatif. Le service vérifie d’abord si une décision portant le même couple organisation/requête a déjà été enregistrée puis réutilise cette décision en cas de replay.

Le moteur peut prendre en compte, selon le scénario :

- second credential ;
- plaque observée et confiance ANPR ;
- politique de rapprochement credential/plaque ;
- terminal ;
- produit ;
- moyen de paiement ;
- montant demandé.

## 9. Isolation multi-tenant

L’isolation tenant est une exigence de sécurité, pas une simple convention d’API.

Pour les lectures engagées :

```text
credential -> WHERE id = ? AND organizationId = ?
entitlement -> WHERE id = ? AND organizationId = ?
list -> WHERE organizationId = ? + filtres
```

Les tests PostgreSQL doivent démontrer qu’un tenant ne peut pas consulter les credentials, droits, quotas ou décisions d’un autre tenant même en connaissant leurs identifiants.

## 10. Cohérence avec le contrat partagé

Le fichier de référence est :

```text
mansa-platform/packages/contracts/src/access-mobility-api.ts
```

Les constantes de routes utilisent désormais explicitement `/v1/internal/access/...`, conformément au contrôleur NestJS réel.

Le test :

```text
mansa-platform/packages/contracts/test/access-mobility-api.test.mjs
```

verrouille le namespace interne et les méthodes HTTP principales afin d’éviter une régression silencieuse vers une surface publique non protégée.

## 11. Prochaine tranche recommandée

L’ordre de construction recommandé est :

1. ajouter pagination par curseur aux listes de credentials et entitlements ;
2. exposer `POST /credentials` et `POST /entitlements` avec idempotence persistée et audit ;
3. exposer `POST /usages` avec replay sûr ;
4. exposer les routes de disponibilité de service et profils terminaux ;
5. exposer l’enregistrement de validation espèces ;
6. ajouter des tests HTTP négatifs sur tenant, identité interne, entrées invalides et replays ;
7. compléter les tests PostgreSQL de concurrence pour les mutations ;
8. remplacer l’identité interne transitoire par une identité de workload attestée avant production.

## 12. Critères de recette de cette tranche

Cette tranche est cohérente lorsque :

- toutes les constantes de route partagées utilisent le namespace interne ;
- le contrôleur et le contrat ne divergent plus sur les routes déjà exécutables ;
- les tests contractuels échouent si une route revient accidentellement sous `/v1/access` ;
- les consultations existantes restent isolées par organisation ;
- aucune route contractuelle non implémentée n’est présentée comme déjà disponible en production.
