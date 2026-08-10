# Recette PostgreSQL des créations de credentials et droits

## 1. Objet

Cette tranche ferme le premier niveau de recette PostgreSQL des mutations internes d’accès et mobilité. Elle complète les tests HTTP unitaires existants en vérifiant la persistance réelle des `AccessCredential` et `AccessEntitlement`, leur isolation multi-tenant et l’écriture atomique de l’audit opérationnel.

Le code de référence se trouve dans :

```text
mansa-platform/apps/api-gateway/test/access-management-postgres.test.mjs
```

La commande de recette PostgreSQL de l’API Gateway exécute désormais ce fichier avec les suites de rapprochement et d’accès déjà présentes.

## 2. Création de credential

La recette vérifie qu’une création valide :

- persiste exactement une ligne `AccessCredentialRecord` ;
- conserve `organizationId`, `subjectId`, le type et la référence publique ;
- conserve uniquement les métadonnées prévues par le modèle ;
- écrit dans la même opération fonctionnelle un `OperationalAuditLog` ;
- relie cet audit au `correlationId` de la commande ;
- conserve la clé d’idempotence dans les métadonnées d’audit sans l’utiliser comme secret.

L’action d’audit attendue est :

```text
ACCESS_CREDENTIAL_CREATED
```

## 3. Rejeu concurrent d’un credential

Deux créations concurrentes de la même ressource ne doivent jamais produire deux credentials.

La recette impose :

```text
même id + même tenant + même référence publique
→ 1 ressource persistée
→ 1 audit de création
→ réponses compatibles avec la ressource initiale
```

Cette protection repose actuellement sur l’identité stable de la ressource, les contraintes uniques PostgreSQL et la reprise après conflit Prisma.

Elle ne remplace pas le futur registre transversal d’idempotence demandé avant exposition à des partenaires externes.

## 4. Références publiques et tenants

Une même référence publique peut exister dans deux organisations différentes lorsque le modèle métier l’autorise, car l’unicité est tenant-scopée avec le type de credential.

En revanche, un identifiant de ressource déjà détenu par un tenant ne peut pas être réutilisé par un autre tenant.

La recette vérifie les deux propriétés afin d’empêcher :

- fuite de données entre organisations ;
- prise de contrôle par collision d’identifiant ;
- faux conflit global sur des tags ou références qui ne doivent être uniques qu’au sein d’un tenant.

## 5. Création d’un entitlement

La recette vérifie la persistance des droits structurés, notamment :

- période de validité ;
- lieux autorisés ;
- produits autorisés ;
- quota d’usage ;
- période de quota ;
- limite monétaire en unités mineures ;
- devise ;
- politique de remboursement ;
- politique de compensation en cas d’indisponibilité ;
- métadonnées métier.

Le montant `150000 XOF` de la fixture est stocké comme entier PostgreSQL/Prisma et reconstruit sans conversion flottante.

L’action d’audit attendue est :

```text
ACCESS_ENTITLEMENT_CREATED
```

## 6. Rejeu et collision d’un entitlement

Le rejeu de la même création ne doit pas dupliquer l’audit.

Un identifiant d’entitlement existant dans une autre organisation doit être rejeté explicitement. Le système ne doit ni retourner la ressource étrangère ni l’écraser.

## 7. Nettoyage des tests

La suite supprime uniquement les audits liés aux actions de création d’accès qu’elle produit puis nettoie les credentials et entitlements de test.

Cette isolation évite que la recette détruise des audits d’autres modules exécutés dans la même base de CI.

## 8. Commande de validation

Dans `apps/api-gateway` :

```bash
pnpm test:postgres
```

La commande exige :

```text
RUN_POSTGRES_TESTS=1
DATABASE_URL=<base PostgreSQL de recette>
```

Aucun identifiant ou secret de production ne doit être utilisé dans cette recette.

## 9. État après cette tranche

Les capacités suivantes disposent maintenant d’une couverture cohérente contrat → HTTP → PostgreSQL :

- lecture de credentials ;
- liste de credentials ;
- création de credentials ;
- lecture de droits ;
- liste de droits ;
- création de droits ;
- évaluation d’accès ;
- isolation multi-tenant ;
- quotas concurrents ;
- audit des créations.

## 10. Prochain lot

Le prochain lot cohérent doit traiter le registre d’idempotence persistant des mutations puis les changements de cycle de vie :

- suspension et révocation de credentials ;
- remplacement de credential ;
- suspension, activation et terminaison d’entitlements ;
- disponibilité de service ;
- profils et états d’affichage des terminaux ;
- usages et validations espèces.

Le registre d’idempotence doit être conçu de façon transversale afin de pouvoir être réutilisé par les autres mutations financières et administratives de Mansa.
