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
7. La prochaine tranche ajoute des tests HTTP et PostgreSQL ciblés sur ces lectures avant d’ouvrir les mutations.

## Fichiers de référence

- `mansa-platform/packages/contracts/src/access-mobility.ts`
- `mansa-platform/packages/contracts/src/access-mobility-api.ts`
- `mansa-platform/apps/api-gateway/src/access/access.controller.ts`
- `mansa-platform/apps/api-gateway/src/access/access.service.ts`
- `mansa-platform/apps/api-gateway/src/access/access.repository.ts`
- `mansa-platform/apps/api-gateway/prisma/schema.prisma`
