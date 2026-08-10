# Registre transversal d’idempotence persistante

## 1. Objet

Cette tranche introduit un registre générique d’idempotence destiné aux mutations sensibles de Mansa. Le composant est transversal afin d’être réutilisé par les domaines accès, paiements, administration, services État et autres opérations à effet de bord.

Implémentation de référence :

```text
mansa-platform/apps/api-gateway/src/idempotency/operation-idempotency.registry.ts
mansa-platform/apps/api-gateway/prisma/migrations/20260810064000_operation_idempotency_registry/migration.sql
```

Le domaine accès orchestre ce registre dans :

```text
mansa-platform/apps/api-gateway/src/access/access.service.ts
```

## 2. Clé fonctionnelle

Une entrée est unique par :

```text
scope + organizationId + idempotencyKey
```

Scopes actifs :

```text
ACCESS_CREDENTIAL_STATUS
ACCESS_ENTITLEMENT_STATUS
ACCESS_CREDENTIAL_REPLACEMENT
```

La même chaîne d’idempotence peut être utilisée dans deux scopes différents sans collision. Elle ne peut pas être réutilisée dans un même scope et tenant avec une charge utile différente.

## 3. Empreinte de requête

Le registre calcule une empreinte SHA-256 de la charge métier. Pour les transitions de statut, elle couvre l’identifiant de ressource, le statut cible et la raison. Pour un remplacement de credential, elle couvre :

- l’identifiant du credential remplacé ;
- le credential de remplacement complet ;
- la raison métier.

Le `correlationId` n’entre pas dans l’empreinte : un rejeu réseau peut porter un nouvel identifiant de corrélation tout en représentant la même commande métier.

Une clé existante associée à une charge différente est rejetée avec :

```text
idempotency key already used with a different payload
```

## 4. Réponse persistée

Une opération réussie enregistre sa réponse JSON avec l’état `COMPLETED`. Un rejeu ultérieur avec la même clé et la même empreinte retourne la réponse initiale persistée, même si l’état courant a ensuite évolué.

Pour le remplacement d’un credential, la réponse persistée contient :

```text
revokedCredential
replacementCredential
```

Le rejeu ne doit jamais créer un second credential de remplacement.

## 5. État PROCESSING et récupération

Une nouvelle clé est d’abord réservée avec `PROCESSING`. L’unicité PostgreSQL protège les appels concurrents.

Pour les transitions, la récupération vérifie que la ressource tenant-scopée se trouve déjà dans le statut cible.

Pour le remplacement, la récupération vérifie simultanément :

- que l’ancien credential appartient au tenant et est `REVOKED` ;
- que le nouveau credential appartient au même tenant et existe sous l’identifiant fourni dans la commande.

Si ces deux conditions sont satisfaites après une interruption située entre l’effet métier et la finalisation du registre, la réponse est reconstruite puis la réservation est marquée `COMPLETED`.

## 6. Remplacement sécurisé d’un credential

Route de référence :

```text
POST /v1/internal/access/credentials/:credentialId/replacement
```

Contrat partagé :

```text
ReplaceAccessCredentialCommand
ReplaceAccessCredentialResult
```

Le workflow réalise dans une seule transaction PostgreSQL :

1. lecture de l’ancien credential ;
2. vérification stricte du tenant ;
3. refus d’un credential déjà `REVOKED` ou `EXPIRED` ;
4. vérification que le nouveau credential appartient au même tenant ;
5. vérification que le `subjectId` reste identique ;
6. contrôle d’unicité de l’identifiant et de la référence publique du nouveau credential ;
7. révocation définitive de l’ancien credential ;
8. création du nouveau credential ;
9. écriture d’un audit `ACCESS_CREDENTIAL_REPLACED` contenant les deux identifiants.

Le lien historique est conservé dans l’audit sous :

```text
replacesCredentialId
replacedByCredentialId
```

Cette approche constitue l’équivalent auditable demandé sans ajouter de colonnes de relation au modèle credential tant que les cas de remplacement en chaîne ne nécessitent pas une navigation SQL directe.

## 7. Concurrence et atomicité

Le registre d’idempotence empêche deux exécutions concurrentes de la même commande. La transaction du repository empêche les états partiels :

- l’ancien credential ne peut pas être révoqué sans création du remplaçant ;
- le nouveau credential ne peut pas être créé sans révocation de l’ancien ;
- l’audit doit réussir dans la même transaction.

Une seconde commande distincte visant à remplacer un credential déjà révoqué est rejetée. Elle ne peut donc pas produire un second remplaçant silencieux.

## 8. Isolation multi-tenant

Le registre est tenant-scopé par `organizationId`. Le workflow refuse explicitement :

- un ancien credential appartenant à une autre organisation ;
- un credential de remplacement dont `organizationId` diffère ;
- une récupération idempotente reposant sur une ressource d’un autre tenant.

Le remplacement impose également le même `subjectId`, afin qu’une opération de sécurité ne puisse pas transférer implicitement un credential vers un autre véhicule, usager ou équipement.

## 9. Persistance du registre

La table `OperationIdempotencyRecord` contient notamment :

- `scope` ;
- `organizationId` ;
- `idempotencyKey` ;
- `requestFingerprint` ;
- `status` ;
- `response` ;
- `correlationId` ;
- `createdAt` ;
- `completedAt`.

Contraintes principales :

```text
UNIQUE(scope, organizationId, idempotencyKey)
status IN (PROCESSING, COMPLETED)
```

## 10. Recette PostgreSQL

Suite dédiée :

```text
mansa-platform/apps/api-gateway/test/idempotency-postgres.test.mjs
```

Elle vérifie désormais :

- rejeu d’une transition sans nouvelle mutation ;
- rejet d’une clé réutilisée avec une charge différente ;
- révocation atomique de l’ancien credential ;
- création unique du remplaçant ;
- rejeu du résultat de remplacement ;
- unicité de l’audit `ACCESS_CREDENTIAL_REPLACED` ;
- présence de `replacesCredentialId` et `replacedByCredentialId` dans l’audit ;
- rejet d’un remplacement cross-tenant ;
- rejet d’un changement de sujet lors du remplacement.

Commande consolidée :

```bash
cd apps/api-gateway
pnpm test:postgres
```

Elle exige une base PostgreSQL de recette et ne doit jamais viser la production.

## 11. Portée actuelle et prochain lot

Le registre protège maintenant :

```text
PATCH /v1/internal/access/credentials/:credentialId/status
PATCH /v1/internal/access/entitlements/:entitlementId/status
POST  /v1/internal/access/credentials/:credentialId/replacement
```

Le prochain lot prioritaire du domaine accès doit porter sur la disponibilité de service et le mode dégradé des points de passage. Le socle de données existe déjà avec `AccessServiceAvailabilityRecord` ; il reste à exposer une mutation idempotente tenant-scopée, son audit, les lectures HTTP et la recette PostgreSQL. Ensuite viennent les profils/états de terminaux, l’enregistrement d’usage et les validations espèces.
