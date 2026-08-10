# Cycle de vie des credentials et droits d’accès

## 1. Objet

Cette tranche ajoute les mutations internes permettant de suspendre, réactiver, révoquer, expirer, annuler ou terminer les ressources du moteur accès et mobilité sans supprimer leur historique.

Les contrats de référence sont :

```text
mansa-platform/packages/contracts/src/access-mobility.ts
mansa-platform/packages/contracts/src/access-mobility-api.ts
```

Les mutations sont implémentées dans :

```text
mansa-platform/apps/api-gateway/src/access/access-management.repository.ts
mansa-platform/apps/api-gateway/src/access/access.service.ts
mansa-platform/apps/api-gateway/src/access/access.controller.ts
```

## 2. Routes internes

Deux routes protégées par l’authentification de service interne complètent les créations et lectures existantes :

```text
PATCH /v1/internal/access/credentials/:credentialId/status
PATCH /v1/internal/access/entitlements/:entitlementId/status
```

Chaque commande exige :

- `organizationId` ;
- identifiant de la ressource dans le chemin ;
- `targetStatus` ;
- `reason` non vide ;
- `idempotencyKey` non vide ;
- `correlationId` non vide.

La mutation reste strictement tenant-scopée. Une ressource appartenant à une autre organisation est traitée comme absente et ne doit jamais être retournée.

## 3. Cycle de vie d’un credential

États :

```text
PENDING
ACTIVE
SUSPENDED
REVOKED
EXPIRED
```

Transitions autorisées :

```text
PENDING   -> ACTIVE | SUSPENDED | REVOKED
ACTIVE    -> SUSPENDED | REVOKED | EXPIRED
SUSPENDED -> ACTIVE | REVOKED | EXPIRED
REVOKED   -> terminal
EXPIRED   -> terminal
```

`REVOKED` et `EXPIRED` sont terminaux. Une carte ou un tag révoqué ne peut donc pas être réactivé par une simple mutation de statut. Un remplacement futur doit créer un nouveau credential et conserver le lien historique avec l’ancien.

## 4. Cycle de vie d’un entitlement

États :

```text
DRAFT
ACTIVE
SUSPENDED
EXPIRED
CANCELLED
TERMINATED
```

Transitions autorisées :

```text
DRAFT     -> ACTIVE | CANCELLED
ACTIVE    -> SUSPENDED | EXPIRED | CANCELLED | TERMINATED
SUSPENDED -> ACTIVE | EXPIRED | CANCELLED | TERMINATED
EXPIRED   -> terminal
CANCELLED -> terminal
TERMINATED-> terminal
```

Une suspension est réversible. Une annulation, une expiration ou une terminaison ferment définitivement la ressource. Un nouveau droit doit être créé si la relation métier doit reprendre après un état terminal.

## 5. Audit opérationnel atomique

Chaque changement réel de statut est écrit dans la même transaction PostgreSQL que son audit.

Actions :

```text
ACCESS_CREDENTIAL_STATUS_CHANGED
ACCESS_ENTITLEMENT_STATUS_CHANGED
```

L’audit conserve au minimum :

- organisation ;
- statut précédent ;
- statut cible ;
- raison ;
- clé d’idempotence fournie ;
- corrélation ;
- ressource concernée.

Un échec d’écriture de l’audit doit annuler le changement de statut.

## 6. Rejeu sans duplication d’audit

Lorsque la ressource est déjà dans le statut cible, le repository retourne l’état courant sans créer un nouvel audit.

Cette propriété rend le rejeu séquentiel de la même transition sans effet supplémentaire.

Elle ne constitue toutefois pas encore un registre d’idempotence transversal : deux commandes différentes qui aboutissent au même état ne sont pas distinguées par persistance de la clé d’idempotence. Le registre persistant transverse reste obligatoire avant exposition de ces mutations à des partenaires externes ou à des flux financiers sensibles.

## 7. Isolation multi-tenant

Les mutations utilisent simultanément :

```text
resourceId + organizationId
```

Une organisation ne peut pas suspendre ou réactiver une ressource d’un autre tenant même si elle connaît son identifiant technique.

Le message d’erreur ne doit pas divulguer le contenu de la ressource étrangère.

## 8. Recette PostgreSQL

La suite dédiée est :

```text
mansa-platform/apps/api-gateway/test/access-status-postgres.test.mjs
```

Elle couvre :

- suspension d’un credential actif ;
- rejeu sans second audit ;
- refus d’une mutation cross-tenant ;
- révocation terminale d’un credential ;
- refus de réactivation après révocation ;
- suspension puis réactivation d’un entitlement ;
- terminaison d’un entitlement ;
- refus de réactivation après terminaison ;
- comptage des audits de transitions.

La commande consolidée reste :

```bash
cd apps/api-gateway
pnpm test:postgres
```

avec une base de recette et jamais des secrets ou données de production.

## 9. Cohérence avec le moteur de décision

Le moteur de décision existant vérifie déjà le statut des credentials et entitlements lors d’une demande d’accès. Une transition vers `SUSPENDED`, `REVOKED`, `EXPIRED`, `CANCELLED` ou `TERMINATED` doit donc être visible lors des évaluations suivantes après lecture de la base.

Les décisions historiques ne sont jamais réécrites : elles restent auditables comme décisions prises avec l’état disponible au moment de leur évaluation.

## 10. Prochain lot cohérent

Le prochain lot prioritaire reste le registre transversal d’idempotence persistant pour les mutations. Il doit ensuite permettre de sécuriser proprement :

- remplacement d’un credential perdu ou compromis ;
- disponibilité de service ;
- profils et états d’affichage des terminaux ;
- enregistrement des usages ;
- validations espèces ;
- mutations administratives et financières des autres modules.

Le registre doit empêcher qu’une même clé soit réutilisée avec une charge utile différente et doit permettre un rejeu déterministe de la réponse initiale.
