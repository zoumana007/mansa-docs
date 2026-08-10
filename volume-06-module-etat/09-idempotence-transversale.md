# Registre transversal d’idempotence persistante

## 1. Objet

Cette tranche introduit un registre générique d’idempotence destiné aux mutations sensibles de Mansa. Le premier branchement concerne les transitions de statut du domaine accès et mobilité, mais la table et le composant sont volontairement transversaux afin d’être réutilisés ensuite par les paiements, l’administration, les services État et les autres opérations à effet de bord.

Implémentation de référence :

```text
mansa-platform/apps/api-gateway/src/idempotency/operation-idempotency.registry.ts
mansa-platform/apps/api-gateway/prisma/migrations/20260810064000_operation_idempotency_registry/migration.sql
```

Le module accès utilise le registre dans :

```text
mansa-platform/apps/api-gateway/src/access/access.service.ts
```

## 2. Clé fonctionnelle

Une entrée est unique par :

```text
scope + organizationId + idempotencyKey
```

Le `scope` sépare les familles d’opérations. Les deux premiers scopes actifs sont :

```text
ACCESS_CREDENTIAL_STATUS
ACCESS_ENTITLEMENT_STATUS
```

La même chaîne d’idempotence peut donc être utilisée dans deux scopes différents sans collision. Elle ne peut pas être réutilisée dans un même scope et tenant avec une charge utile différente.

## 3. Empreinte de requête

Le registre calcule une empreinte SHA-256 de la charge métier normalisée. Pour les transitions d’accès, l’empreinte couvre :

- identifiant de la ressource ;
- statut cible ;
- raison métier.

Le `correlationId` n’entre pas dans cette empreinte : un rejeu réseau peut recevoir un nouvel identifiant de corrélation tout en représentant exactement la même commande métier.

Si une clé existante porte une autre empreinte, la commande est rejetée avec un conflit explicite :

```text
idempotency key already used with a different payload
```

Cette règle empêche qu’une clé déjà consommée serve ensuite à demander une autre mutation.

## 4. Réponse persistée

Une opération réussie enregistre sa réponse JSON dans le registre avec l’état :

```text
COMPLETED
```

Un rejeu ultérieur avec la même clé et la même empreinte retourne cette réponse persistée, même si l’état courant de la ressource a évolué depuis sous l’effet d’une autre commande.

Ce comportement est important : l’idempotence rejoue le résultat de la commande initiale et ne transforme pas le rejeu en nouvelle lecture de l’état courant.

## 5. État PROCESSING

Une nouvelle clé est d’abord réservée avec :

```text
PROCESSING
```

L’unicité PostgreSQL protège les traitements concurrents. Une seule tentative peut créer la réservation pour un même triplet `scope + organizationId + idempotencyKey`.

Pour les transitions de statut, un mécanisme de récupération complète les rejeux interrompus : si une entrée reste `PROCESSING` mais que la ressource est déjà dans le statut cible, le service reconstruit la réponse depuis l’état tenant-scopé, marque l’entrée `COMPLETED` puis retourne cette réponse.

Cette récupération couvre notamment une interruption située après l’effet métier mais avant la persistance finale de la réponse d’idempotence.

## 6. Échec avant effet durable

Si l’opération métier échoue, la réservation `PROCESSING` est supprimée. Un appel corrigé ou un retry peut alors reprendre avec la même clé, à condition que la charge utile corresponde au comportement attendu par le client.

Les opérations qui seront branchées ultérieurement doivent définir une stratégie de récupération adaptée lorsque leur effet est observable dans un autre registre durable.

## 7. Isolation multi-tenant

Le registre est tenant-scopé par `organizationId`. Une clé utilisée par une organisation ne bloque pas la même clé chez une autre organisation.

La récupération d’une opération accès passe également par les lectures tenant-scopées du repository d’accès. Elle ne doit jamais lire une ressource d’un autre tenant pour conclure qu’une opération a réussi.

## 8. Persistance PostgreSQL

La migration crée :

```text
OperationIdempotencyRecord
```

Champs principaux :

- `scope` ;
- `organizationId` ;
- `idempotencyKey` ;
- `requestFingerprint` ;
- `status` ;
- `response` ;
- `correlationId` ;
- `createdAt` ;
- `completedAt`.

Contraintes :

```text
UNIQUE(scope, organizationId, idempotencyKey)
status IN (PROCESSING, COMPLETED)
```

Des index existent également sur l’organisation/date et la corrélation pour les besoins d’exploitation et d’audit.

## 9. Recette PostgreSQL

Suite dédiée :

```text
mansa-platform/apps/api-gateway/test/idempotency-postgres.test.mjs
```

Elle vérifie notamment :

- rejeu de la réponse initiale avec la même clé et la même charge ;
- absence de nouvelle mutation pendant le rejeu ;
- conservation du résultat historique même si la ressource a ensuite changé d’état ;
- rejet d’une réutilisation de la même clé avec une charge différente.

La commande consolidée est :

```bash
cd apps/api-gateway
pnpm test:postgres
```

Elle exige une base PostgreSQL de recette et ne doit jamais viser la production.

## 10. Portée actuelle et prochain lot

Le composant est générique mais le branchement opérationnel initial couvre uniquement :

```text
PATCH /v1/internal/access/credentials/:credentialId/status
PATCH /v1/internal/access/entitlements/:entitlementId/status
```

Le prochain lot cohérent doit réutiliser ce registre pour le remplacement d’un credential perdu ou compromis. Ce workflow devra :

1. révoquer définitivement l’ancien credential ;
2. créer un nouveau credential ;
3. conserver le lien `replacesCredentialId` / `replacedByCredentialId` ou un équivalent auditable ;
4. être protégé par une seule commande idempotente ;
5. préserver l’isolation multi-tenant ;
6. empêcher deux remplacements concurrents du même credential ;
7. couvrir le scénario par une recette PostgreSQL.

Après ce remplacement sécurisé, les lots prioritaires restent la disponibilité de service, les profils et états de terminaux, l’enregistrement d’usage et les validations espèces avant extension aux mutations financières et administratives des autres modules.
