# Identité et autorisation des workloads internes

## Objectif

Les routes internes Mansa ne doivent pas faire confiance à un identifiant d’organisation fourni librement dans une query, un body ou un header applicatif. La portée d’organisation et les permissions doivent provenir d’une identité de workload vérifiée cryptographiquement par un mécanisme approuvé.

Cette tranche complète la protection historique `InternalServiceGuard` et prépare son remplacement progressif sur les routes ledger, rapprochement et opérations.

## Contrat partagé

Le contrat de référence est `@mansa/contracts/workload-identity` dans `mansa-platform/packages/contracts/src/workload-identity.ts`.

Une identité normalisée contient :

- `version` ;
- `workloadId` stable ;
- `organizationId` ;
- liste de `scopes` ;
- `issuedAt` ;
- `expiresAt` ;
- `tokenId` unique.

La durée maximale acceptée est de 15 minutes. Les identités expirées, futures au-delà de la tolérance, mal formées, sans scope ou contenant un scope non supporté sont rejetées.

## Scopes initiaux

Les scopes initiaux sont :

```text
ledger:read
ledger:write
reconciliation:read
reconciliation:write
operations:read
operations:write
```

Les routes de lecture ne doivent jamais exiger un scope d’écriture uniquement pour simplifier la configuration. Les opérations mutantes exigent un scope d’écriture explicite.

## Frontière de vérification

`WorkloadIdentityVerifier` définit la frontière technique de vérification de la credential brute. Une implémentation de production pourra utiliser notamment :

- OIDC avec issuer et audience contrôlés et clés JWKS ;
- mTLS avec identité de certificat contrôlée ;
- SPIFFE/SPIRE ou mécanisme équivalent d’identité de workload.

Le mécanisme retenu doit être documenté par ADR avant activation en production.

La credential brute ne doit jamais être persistée ni journalisée.

## Guard d’authentification

`WorkloadIdentityGuard` :

1. exige un bearer credential ;
2. délègue la vérification brute au `WorkloadIdentityVerifier` ;
3. applique `validateWorkloadIdentity` ;
4. transforme l’identité validée en `WorkloadIdentityContext` ;
5. attache uniquement le contexte normalisé à la requête ;
6. renvoie une erreur générique en cas d’échec sans exposer les détails du fournisseur ni la credential.

Le contexte contient uniquement :

```text
workloadId
organizationId
scopes
tokenId
```

## Guard d’autorisation

`WorkloadScopeGuard` complète l’authentification. Les contrôleurs déclarent les scopes requis avec `RequireWorkloadScopes(...)`.

Le comportement est volontairement fail-closed :

- aucune politique de scope déclarée → refus ;
- contexte workload absent → refus ;
- un seul scope requis manquant → refus ;
- tous les scopes requis présents → autorisation.

Cette règle empêche qu’une nouvelle route interne soit exposée sans politique explicite.

## Portée organisationnelle

Pour les repositories multi-tenant, `organizationId` doit provenir du contexte workload attesté et être injecté dans les requêtes Prisma elles-mêmes.

Le schéma cible est :

```text
credential signée
→ vérification cryptographique
→ identité workload normalisée
→ organisation attestée + scopes
→ repository scoppé organisation
→ réponse présentée sans champs internes
```

Un `organizationId` reçu depuis l’appelant ne doit pas pouvoir élargir la portée de l’identité attestée.

## Migration du rapprochement

État actuel : le repository de rapprochement applique déjà l’isolation tenant au niveau Prisma, mais le contrôleur demande encore `organizationId` à l’appelant derrière `InternalServiceGuard`.

Migration prévue :

1. fournir une implémentation de production de `WorkloadIdentityVerifier` ;
2. enregistrer le verifier et les deux guards dans le module NestJS ;
3. appliquer `WorkloadIdentityGuard` puis `WorkloadScopeGuard` ;
4. supprimer `organizationId` des paramètres publics des routes internes ;
5. utiliser `request.workloadIdentity.organizationId` ;
6. appliquer `reconciliation:read` aux lectures ;
7. appliquer `reconciliation:write` aux résolutions/imports ;
8. couvrir les tentatives inter-tenant et scopes insuffisants en tests HTTP et PostgreSQL.

La migration ne doit pas être activée avec un verifier factice permissif.

## Journalisation et corrélation

Les événements d’audit peuvent conserver :

- `workloadId` ;
- `organizationId` ;
- `tokenId` comme identifiant de corrélation de credential, sans token brut ;
- `correlationId` de la requête ;
- action et ressource concernées.

Aucun JWT, bearer token, certificat privé ou secret ne doit être écrit dans les logs.

## Critères de recette

La tranche est considérée prête pour intégration lorsque :

- le contrat partagé est couvert par tests ;
- le guard d’authentification rejette credential absente, invalide et identité expirée ;
- le guard de scopes échoue si aucune politique n’est déclarée ;
- un workload de lecture ne peut pas exécuter une mutation ;
- un workload d’une organisation ne peut pas lire ou modifier les données d’une autre ;
- les erreurs ne révèlent pas le contenu de la credential ;
- les routes migrées ne dépendent plus d’un `organizationId` fourni librement par l’appelant ;
- le verifier de production contrôle issuer/audience ou identité mTLS équivalente ;
- la rotation des clés/certificats est documentée et testée.

## État d’implémentation

Implémenté dans `mansa-platform` :

- contrat et validation `WorkloadIdentity` ;
- abstraction `WorkloadIdentityVerifier` ;
- `WorkloadIdentityGuard` ;
- tests du guard d’authentification ;
- `WorkloadScopeGuard` et décorateur `RequireWorkloadScopes` ;
- tests fail-closed des scopes.

Restent à réaliser : verifier de production, câblage NestJS, migration des contrôleurs internes, suppression des portées fournies par l’appelant, tests d’intégration et procédures de rotation/révocation.
