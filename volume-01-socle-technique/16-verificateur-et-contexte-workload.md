# Vérificateur et contexte d’identité workload

## 1. Objet

Cette note décrit la première tranche d’implémentation de la phase B de l’identité workload attestée. Elle complète `15-identite-workload-et-portee-tenant.md` sans activer encore cette identité sur les routes existantes.

L’objectif est de créer une frontière technique stable entre :

- le credential brut présenté par un service ;
- la vérification cryptographique fournie plus tard par OIDC/JWKS, mTLS/SPIFFE ou un mécanisme approuvé ;
- le contexte normalisé utilisé par les contrôleurs et repositories.

## 2. Éléments ajoutés dans `mansa-platform`

Le package `@mansa/contracts` expose désormais explicitement le sous-chemin public :

```text
@mansa/contracts/workload-identity
```

Le backend ajoute :

```text
apps/api-gateway/src/workload-identity.verifier.ts
apps/api-gateway/src/workload-identity.guard.ts
apps/api-gateway/test/workload-identity.guard.test.mjs
```

`WorkloadIdentityVerifier` est une abstraction. Il reçoit un credential brut et doit retourner une `WorkloadIdentity` vérifiée. Aucun vérificateur de production n’est fourni par défaut : il est interdit de traiter un simple JSON décodé comme une preuve attestée.

`WorkloadIdentityGuard` :

1. exige un credential `Authorization: Bearer ...` ;
2. transmet uniquement le credential au vérificateur injecté ;
3. réapplique la validation structurelle partagée après vérification ;
4. transforme l’identité en `WorkloadIdentityContext` ;
5. attache uniquement ce contexte normalisé à la requête ;
6. transforme les erreurs fournisseur en refus générique sans exposer les détails internes ni le credential.

## 3. Données autorisées dans le contexte

Le contexte applicatif contient uniquement :

```text
workloadId
organizationId
scopes
tokenId
```

Les champs `issuedAt`, `expiresAt`, signature, certificat, bearer token ou preuve complète restent hors des repositories et du contexte métier courant.

## 4. Propriétés de sécurité

La tranche respecte les règles suivantes :

- aucune clé ni aucun token réel dans Git ;
- aucune implémentation permissive par défaut ;
- absence de credential = `401` ;
- erreur du vérificateur = `401` générique ;
- identité expirée ou structurellement invalide = refus ;
- le credential brut n’est pas attaché à la requête ;
- la portée organisationnelle vient de l’identité vérifiée ;
- les repositories devront continuer d’appliquer `organizationId` dans leurs requêtes SQL/Prisma.

## 5. Tests ajoutés

Les tests couvrent :

- absence de Bearer ;
- propagation exacte du credential au vérificateur ;
- création du contexte normalisé ;
- absence de copie du credential sur la requête ;
- masquage d’une erreur interne du fournisseur ;
- rejet d’une identité expirée ;
- gestion d’un header représenté sous forme de tableau par le runtime.

## 6. Ce qui n’est pas encore activé

Cette tranche est volontairement additive. `InternalServiceGuard` reste le guard utilisé par les routes existantes tant que la chaîne cryptographique réelle n’est pas disponible et testée.

Il reste notamment à construire :

1. un vérificateur OIDC/JWKS ou SPIFFE réellement attesté ;
2. validation stricte issuer/audience et politique de clés ;
3. cache et rotation des clés publiques ;
4. configuration fail-closed par environnement ;
5. décorateurs/scopes de route et refus `403` pour scope insuffisant ;
6. métriques d’acceptation/refus sans fuite de credential ;
7. migration du rapprochement vers `request.workloadIdentity.organizationId` ;
8. suppression de `organizationId` comme source d’autorité dans query/body ;
9. migration du ledger et des routes opérationnelles ;
10. suppression définitive du token statique en production.

## 7. Critère de passage à la tranche suivante

La prochaine tranche peut commencer lorsque le backend compile avec le sous-chemin `@mansa/contracts/workload-identity` et que les tests du nouveau guard sont verts. La prochaine étape prioritaire est le mécanisme de scopes de route, qui peut être développé et testé avec un faux vérificateur sans prétendre disposer déjà d’une attestation cryptographique de production.
