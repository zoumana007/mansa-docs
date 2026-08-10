# Gestion interne des credentials et droits d’accès

## 1. Objet

Cette tranche complète le moteur d’accès et mobilité avec les premières mutations de gestion réellement exposées par l’API Gateway. Elle couvre la création d’un `AccessCredential` et d’un `AccessEntitlement` sans dépendre du fabricant du lecteur, de la borne ou du support physique.

Les routes sont internes et protégées par le même garde de service que les lectures et l’évaluation d’accès.

## 2. Routes de référence

Le contrat partagé `mansa-platform/packages/contracts/src/access-mobility-api.ts` définit :

```text
POST /v1/internal/access/credentials
POST /v1/internal/access/entitlements
GET  /v1/internal/access/credentials/:credentialId
GET  /v1/internal/access/credentials
GET  /v1/internal/access/entitlements/:entitlementId
GET  /v1/internal/access/entitlements
POST /v1/internal/access/evaluate
```

Aucune de ces routes n’est une API publique destinée directement à une borne ou à une application mobile. Les interfaces externes devront passer par une couche d’autorisation et de présentation adaptée à leur contexte.

## 3. Création d’un credential

La commande contient :

- `credential` ;
- `idempotencyKey` ;
- `correlationId`.

Le credential doit au minimum porter :

- un identifiant stable ;
- l’organisation propriétaire ;
- le type de sujet ;
- le sujet ;
- le type de credential ;
- une référence publique ;
- un statut.

La référence publique n’est pas un secret. Aucun secret RFID, clé de carte, PIN, donnée biométrique ou clé cryptographique de production ne doit être stocké dans ce modèle.

La persistance impose déjà l’unicité de `(organizationId, credentialType, publicReference)`. Une collision avec une autre ressource est refusée. Un rejeu de la même identité de ressource peut retourner la ressource existante au lieu de créer un doublon.

## 4. Création d’un entitlement

Le droit est indépendant du credential physique. La commande contient :

- `entitlement` ;
- `idempotencyKey` ;
- `correlationId`.

Les validations initiales couvrent au minimum :

- organisation et sujet obligatoires ;
- cas d’usage et statut obligatoires ;
- `validFrom` valide ;
- `validUntil` postérieur ou égal à `validFrom` lorsqu’il existe ;
- quota d’usage positif lorsqu’il existe ;
- montant stocké en unités mineures entières ;
- listes de lieux et produits conservées comme données structurées.

Remplacer une carte, un tag ou un QR ne doit pas imposer de recréer ce droit.

## 5. Audit et corrélation

Une création effective écrit un `OperationalAuditLog` dans la même transaction PostgreSQL que la ressource créée.

Actions initiales :

```text
ACCESS_CREDENTIAL_CREATED
ACCESS_ENTITLEMENT_CREATED
```

L’audit conserve :

- `correlationId` ;
- type de ressource ;
- identifiant de ressource ;
- organisation ;
- sujet ;
- clé d’idempotence fournie à l’opération.

Le lot actuel représente l’appelant par l’identité technique `internal-service`. L’identité de workload attestée devra remplacer cette identité transitoire avant production.

## 6. Isolation multi-tenant

Les lectures restent obligatoirement filtrées par `organizationId`. Pour les créations :

- l’organisation est portée par la ressource ;
- une ressource existante d’un autre tenant ne peut pas être réutilisée ;
- une référence publique unique n’est comparée qu’avec son organisation et son type ;
- les réponses ne doivent pas exposer les données d’un autre tenant lors d’un conflit.

Les prochaines mutations de suspension, révocation, remplacement et modification de droit devront appliquer la même règle.

## 7. Idempotence

Le contrat exige une `idempotencyKey` non vide sur les créations. La tranche actuelle protège déjà les replays les plus dangereux par l’identité stable de la ressource et les contraintes uniques PostgreSQL, y compris les courses de création.

Avant la mise en production, la plateforme doit compléter ce mécanisme par un registre d’idempotence persistant liant explicitement :

```text
organisation + opération + idempotencyKey -> empreinte de requête + résultat
```

Une même clé avec une empreinte différente devra être refusée. Cette amélioration est obligatoire avant d’exposer des mutations à des systèmes partenaires non fiables.

## 8. Tests minimaux

La recette de cette tranche doit vérifier :

1. rejet d’une commande sans clé d’idempotence ;
2. rejet d’une commande sans corrélation ;
3. validation des champs obligatoires du credential ;
4. validation de `validFrom` pour un droit ;
5. refus d’une période inversée ;
6. création sans doublon sur rejeu ;
7. refus d’une collision inter-tenant ;
8. écriture atomique de l’audit avec la création ;
9. conservation des lectures multi-tenant existantes ;
10. maintien des tests d’évaluation et de quota existants.

Les tests HTTP unitaires sont engagés dans `mansa-platform/apps/api-gateway/test/access-controller.test.mjs`. Les validations PostgreSQL des mutations devront être ajoutées au lot d’intégration existant.

## 9. Prochain lot

Le prochain lot cohérent doit couvrir :

- tests PostgreSQL des créations et de leurs audits ;
- registre d’idempotence persistant ou mécanisme transversal équivalent ;
- suspension/révocation/remplacement d’un credential ;
- suspension/activation/terminaison d’un entitlement ;
- mise à jour de disponibilité de service ;
- lecture et mise à jour des profils de terminaux ;
- enregistrement des validations espèces et usages ;
- identité de service attestée et RBAC interne plus fin.
