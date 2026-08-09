# API interne — Accès, mobilité et cartes multiservices

## Objet

Cette tranche documente l’exposition interne du moteur d’accès dans `mansa-platform/apps/api-gateway`. Elle complète le contrat partagé `@mansa/contracts/access-mobility-api` sans remplacer les futures routes publiques `/v1/access/*`.

Le premier objectif est de permettre aux services de confiance de consulter les identifiants d’accès et droits persistés, puis d’évaluer une demande d’accès, tout en conservant l’isolation stricte par organisation.

## Routes internes actuellement exposées

Base NestJS : `/v1/internal/access` lorsque le versioning URI de l’API Gateway est actif.

| Méthode | Route | Objet |
|---|---|---|
| `GET` | `/credentials/:credentialId?organizationId=...` | Lire un identifiant d’accès dans le tenant demandé |
| `GET` | `/credentials?organizationId=...` | Lister les identifiants d’un tenant |
| `GET` | `/entitlements/:entitlementId?organizationId=...` | Lire un droit d’accès dans le tenant demandé |
| `GET` | `/entitlements?organizationId=...` | Lister les droits d’un tenant |
| `POST` | `/evaluate` | Évaluer une demande d’accès et journaliser la décision |

Toutes ces routes sont protégées par `InternalServiceGuard`. Cette protection reste transitoire tant que l’identité de workload attestée n’est pas branchée.

## Isolation multi-tenant

`organizationId` est obligatoire pour chaque lecture. Le repository Prisma applique simultanément l’identifiant de ressource et l’organisation : une ressource existante dans une autre organisation doit donc être indistinguable d’une ressource absente et produire `404` au niveau HTTP.

Une liste ne peut jamais retourner des enregistrements d’une autre organisation. Les filtres métier ne remplacent jamais le filtre tenant.

## Consultation des identifiants

Filtres pris en charge :

- `subjectId` ;
- `status` ;
- `credentialType` ;
- `limit` compris entre `1` et `100`.

L’ordre de référence est `createdAt DESC, id DESC`. Cette tranche borne volontairement la lecture à 100 éléments. Le curseur opaque défini par le contrat partagé doit être ajouté avant exposition publique.

Les valeurs retournées sont reconstruites en `AccessCredential`; les métadonnées ne conservent que les paires chaîne/chaîne attendues par le contrat.

## Consultation des droits

Filtres pris en charge :

- `subjectId` ;
- `useCase` ;
- `status` ;
- `limit` compris entre `1` et `100`.

L’ordre de référence est `validFrom DESC, id DESC`. Les montants sont reconstruits à partir des unités mineures `BigInt` PostgreSQL vers la représentation chaîne de `Money`.

## Validations automatisées

La tranche de lecture est désormais couverte à deux niveaux.

### Contrôleur HTTP interne

`apps/api-gateway/test/access-controller.test.mjs` vérifie :

- rejet d’une lecture sans `organizationId` avant tout appel au service ;
- conversion d’une ressource absente dans le tenant demandé en `404` ;
- propagation correcte du tenant et des filtres combinés ;
- validation de `limit` entre `1` et `100` ;
- absence d’appel à la persistance lorsqu’une requête est invalide.

### PostgreSQL / Prisma

`apps/api-gateway/test/access-postgres.test.mjs` couvre maintenant aussi :

- lecture d’un credential limitée à son organisation ;
- non-divulgation du même identifiant depuis un autre tenant ;
- filtrage combiné `subjectId + status + credentialType` sans fuite cross-tenant ;
- lecture d’un entitlement limitée à son organisation ;
- reconstruction d’un `amountLimit` depuis `BIGINT` vers `Money.amountMinor` chaîne ;
- filtrage combiné `subjectId + useCase + status` sans fuite cross-tenant ;
- conservation des tests de concurrence, idempotence de décision et isolation des quotas déjà présents.

Les scénarios PostgreSQL restent opt-in localement via `RUN_POSTGRES_TESTS=1` et sont exécutés par la CI PostgreSQL dédiée configurée dans le dépôt plateforme.

## Erreurs

- `400` : `organizationId` absent/vide ou `limit` hors limites ;
- `404` : ressource non trouvée dans l’organisation demandée ;
- `401/403` : rejet par la protection de service interne ;
- les erreurs de persistance inattendues ne doivent pas être transformées en succès ou en liste vide.

## Écart volontaire avec le contrat public

Le contrat partagé prévoit aussi : création d’identifiant, création de droit, enregistrement d’usage, disponibilité de service, profil terminal, état d’affichage et validation espèces.

Ces opérations ne doivent pas être simulées tant que leurs règles d’idempotence, d’audit, de mutation et d’autorisation ne sont pas complètement persistées. En particulier, les commandes de création comportent une `idempotencyKey`; la future implémentation doit conserver durablement cette clé ou une empreinte équivalente avant de rendre la création disponible.

## Critères d’acceptation de la tranche

1. Toute lecture exige `organizationId`.
2. Une ressource d’un autre tenant n’est jamais retournée.
3. Les listes sont bornées à 100 éléments.
4. Les filtres restent combinables avec l’isolation tenant.
5. L’évaluation existante conserve son idempotence par `(organizationId, requestId)`.
6. Aucune donnée secrète ou credential technique privée n’est exposée : seul `publicReference` appartient au contrat.
7. Les lectures sont couvertes par des tests de contrôleur et PostgreSQL avant l’ouverture des mutations.
8. La prochaine tranche doit persister l’idempotence et l’audit des créations avant d’exposer `createCredential` et `createEntitlement`.

## Fichiers de référence

- `mansa-platform/packages/contracts/src/access-mobility.ts`
- `mansa-platform/packages/contracts/src/access-mobility-api.ts`
- `mansa-platform/apps/api-gateway/src/access/access.controller.ts`
- `mansa-platform/apps/api-gateway/src/access/access.service.ts`
- `mansa-platform/apps/api-gateway/src/access/access.repository.ts`
- `mansa-platform/apps/api-gateway/test/access-controller.test.mjs`
- `mansa-platform/apps/api-gateway/test/access-postgres.test.mjs`
- `mansa-platform/apps/api-gateway/prisma/schema.prisma`
