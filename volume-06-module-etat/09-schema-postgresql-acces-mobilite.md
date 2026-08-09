# Schéma PostgreSQL initial — accès et mobilité

## 1. Objet

Cette tranche matérialise dans `mansa-platform` la première persistance PostgreSQL/Prisma du moteur d’accès et de mobilité.

Elle couvre les données nécessaires aux ports applicatifs déjà définis :

- credentials d’accès ;
- droits/entitlements ;
- disponibilité de service ;
- profil matériel de borne ;
- décisions ;
- usages ;
- compteurs de quota ;
- réservations idempotentes de quota.

Les modèles Prisma se trouvent dans :

`apps/api-gateway/prisma/schema.prisma`

La migration SQL associée se trouve dans :

`apps/api-gateway/prisma/migrations/20260809194000_access_mobility_persistence/migration.sql`

## 2. Isolation multi-tenant

Toutes les données fonctionnelles d’accès sont scindées par `organizationId`.

Les contraintes uniques importantes incluent explicitement ce tenant :

```text
credential     = organizationId + credentialType + publicReference
decision       = organizationId + requestId
usage          = organizationId + requestId
quota counter  = organizationId + entitlementId + periodStart
reservation    = organizationId + entitlementId + periodStart + requestId
```

Deux organisations peuvent donc utiliser le même identifiant métier ou le même `requestId` sans collision.

## 3. Credentials

`AccessCredentialRecord` stocke l’identifiant public utilisé à la borne sans supposer un seul support.

Il couvre notamment :

- carte NFC ;
- tag RFID UHF ;
- QR ;
- plaque ;
- jeton de terminal/appareil.

La contrainte `(organizationId, credentialType, publicReference)` interdit les doublons dans une même organisation tout en conservant l’isolation entre exploitants.

## 4. Entitlements

`AccessEntitlementRecord` porte le droit accordé au sujet :

- cas d’usage ;
- statut ;
- fenêtre de validité ;
- lieux autorisés ;
- produits autorisés ;
- quota et période ;
- limite monétaire facultative ;
- politique de remboursement ;
- politique de compensation en cas de panne.

Les listes extensibles sont initialement stockées en JSON pour éviter de figer prématurément un modèle relationnel avant les premiers parcours complets. Elles pourront être normalisées plus tard si les besoins d’indexation ou de reporting le justifient.

## 5. Disponibilité et profil de borne

`AccessServiceAvailabilityRecord` représente l’état opérationnel d’un lieu ou d’une voie.

`AccessTerminalProfileRecord` représente les capacités physiques d’une borne :

- hauteur simple basse, haute ou double hauteur ;
- carte/NFC ;
- QR ;
- Mobile Money ;
- billets ;
- pièces ;
- rendu de monnaie ;
- imprimante de reçu ;
- interphone ;
- devises et coupures supportées.

Le champ technique `laneKey` utilise une chaîne vide pour représenter l’absence de voie explicite. Cela évite les ambiguïtés des contraintes uniques PostgreSQL contenant une valeur `NULL`.

## 6. Décisions et usages idempotents

`AccessDecisionRecord` impose une seule décision par `(organizationId, requestId)`.

`AccessUsageRecord` impose un seul usage par `(organizationId, requestId)`.

Une répétition réseau ne peut donc pas créer plusieurs décisions ou consommations pour la même requête logique dans un tenant.

Les montants sont séparés en :

```text
amountMinor
currency
```

Aucun montant financier n’est stocké en nombre flottant.

## 7. Compteur et réservation de quota

`AccessQuotaCounter` matérialise le quota d’un entitlement sur une fenêtre UTC donnée.

Il conserve :

- `periodStart` ;
- `periodEnd` ;
- `used` ;
- `limit`.

La migration ajoute une contrainte SQL :

```text
used >= 0
limit >= 0
used <= limit
```

`AccessQuotaReservationRecord` représente la réservation idempotente d’une requête dans cette fenêtre.

La contrainte unique correspond exactement à l’identité logique définie dans la tranche précédente :

```text
organizationId + entitlementId + periodStart + requestId
```

## 8. Concurrence

Le schéma rend possible l’algorithme atomique suivant :

1. création ou chargement du compteur de période ;
2. `UPDATE` conditionnel du compteur uniquement si `used < limit` ;
3. insertion de la réservation dans la même transaction ;
4. rollback complet en cas d’échec ;
5. replay d’une réservation existante traité comme idempotent, pas comme une nouvelle consommation.

La prochaine tranche doit implémenter cet algorithme dans le repository Prisma et le démontrer avec deux transactions PostgreSQL concurrentes visant la dernière place disponible.

## 9. Contrats publiés

Les sous-chemins suivants sont maintenant publiés explicitement par `@mansa/contracts` :

- `@mansa/contracts/access-application-service` ;
- `@mansa/contracts/access-decision-engine` ;
- `@mansa/contracts/access-persistence`.

Cela permet à l’API Gateway d’utiliser les ports et helpers sans dépendre de chemins internes au package.

## 10. Ce qui reste à implémenter

Cette tranche fournit le schéma et la migration, mais ne prétend pas terminer la persistance applicative.

La tranche suivante doit encore fournir :

- `PrismaAccessApplicationRepository` ;
- `PrismaAccessQuotaReservation` ;
- `PrismaAccessDecisionJournal` ;
- transaction atomique compteur + réservation ;
- relecture idempotente d’une décision existante ;
- tests PostgreSQL réels d’isolation entre tenants ;
- test concurrent du dernier quota ;
- branchement du service applicatif dans le module HTTP interne.
