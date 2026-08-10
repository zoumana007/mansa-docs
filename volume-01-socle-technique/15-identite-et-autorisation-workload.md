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

`WorkloadIdentityVerifier` définit la frontière technique de vérification de la credential brute.

Une première implémentation cryptographique stricte existe désormais : `HmacWorkloadIdentityVerifier`. Elle vérifie un JWT HS256 signé avec un secret d’au moins 32 octets, impose `issuer` et `audience`, refuse les algorithmes inattendus et utilise une comparaison de signature résistante aux attaques temporelles.

Cette implémentation est adaptée au socle interne contrôlé et aux environnements de développement/intégration. Pour une architecture de production distribuée, la cible reste un mécanisme dont la rotation et la révocation ne reposent pas sur un secret symétrique partagé globalement, par exemple :

- OIDC avec issuer et audience contrôlés et clés JWKS ;
- mTLS avec identité de certificat contrôlée ;
- SPIFFE/SPIRE ou mécanisme équivalent d’identité de workload.

Le mécanisme définitif doit être documenté par ADR avant généralisation en production.

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

La migration du contrôleur de rapprochement est désormais réalisée sur les routes de consultation et de résolution :

- `WorkloadIdentityGuard` et `WorkloadScopeGuard` sont câblés dans le module NestJS ;
- `HmacWorkloadIdentityVerifier` est enregistré derrière `WORKLOAD_IDENTITY_VERIFIER` ;
- les routes de lecture exigent `reconciliation:read` ;
- la résolution exige `reconciliation:write` ;
- l’organisation provient de `request.workloadIdentity.organizationId` ;
- les routes ne dépendent plus d’un `organizationId` fourni librement par l’appelant ;
- l’acteur d’une résolution provient du `workloadId` attesté ;
- le repository continue d’appliquer la portée tenant directement dans les requêtes Prisma.

Les tests couvrent le guard, les scopes, la dérivation du tenant depuis l’identité attestée et la séparation organisationnelle déjà validée au niveau repository/PostgreSQL.

## Journalisation et corrélation

Les événements d’audit peuvent conserver :

- `workloadId` ;
- `organizationId` ;
- `tokenId` comme identifiant de corrélation de credential, sans token brut ;
- `correlationId` de la requête ;
- action et ressource concernées.

Aucun JWT, bearer token, certificat privé ou secret ne doit être écrit dans les logs.

## Rotation, révocation et anti-rejeu

La présence de `tokenId` permet d’identifier de manière stable une credential émise, mais le socle actuel ne fournit pas encore de registre distribué de révocation ou de détection de rejeu.

Avant généralisation en production, il faut ajouter :

- rotation des clés sans interruption ;
- mécanisme de révocation rapide d’un workload ou d’une clé ;
- stratégie anti-rejeu adaptée aux opérations mutantes sensibles ;
- stockage distribué ou mécanisme équivalent lorsque plusieurs instances d’API Gateway sont actives ;
- métriques sur refus d’authentification, scopes insuffisants, issuer/audience invalides et credentials expirées ;
- alerte sur hausse anormale des refus ou tentative de réutilisation.

Un simple cache mémoire local ne doit pas être présenté comme protection anti-rejeu complète dans un déploiement horizontal.

## Critères de recette

La tranche runtime actuelle est considérée intégrée lorsque :

- le contrat partagé est couvert par tests ;
- le guard d’authentification rejette credential absente, invalide et identité expirée ;
- le guard de scopes échoue si aucune politique n’est déclarée ;
- un workload de lecture ne peut pas exécuter une mutation ;
- un workload d’une organisation ne peut pas lire ou modifier les données d’une autre ;
- les erreurs ne révèlent pas le contenu de la credential ;
- les routes migrées ne dépendent plus d’un `organizationId` fourni librement par l’appelant ;
- le verifier contrôle issuer, audience, algorithme et signature ;
- le comportement HMAC est couvert par tests de signature valide, altération, algorithme non supporté, issuer/audience invalides et configuration faible.

Pour la production distribuée, restent des critères supplémentaires : rotation/révocation, anti-rejeu distribué, observabilité de sécurité et mécanisme d’identité de workload définitif validé par ADR.

## État d’implémentation

Implémenté dans `mansa-platform` :

- contrat et validation `WorkloadIdentity` ;
- abstraction `WorkloadIdentityVerifier` ;
- `HmacWorkloadIdentityVerifier` avec validation cryptographique stricte HS256, issuer et audience ;
- tests unitaires du verifier HMAC ;
- `WorkloadIdentityGuard` ;
- tests du guard d’authentification ;
- `WorkloadScopeGuard` et décorateur `RequireWorkloadScopes` ;
- tests fail-closed des scopes ;
- câblage NestJS du verifier et des guards dans le module rapprochement ;
- migration des routes de rapprochement vers l’organisation attestée ;
- scopes `reconciliation:read` et `reconciliation:write` ;
- acteur de résolution dérivé du `workloadId` attesté ;
- tests de tenant dérivé du workload.

Restent à réaliser :

1. généraliser progressivement la même identité workload aux autres contrôleurs internes, notamment Ledger et opérations ;
2. définir l’ADR du mécanisme d’identité de production ;
3. mettre en place rotation/révocation et anti-rejeu distribué pour les opérations sensibles ;
4. exposer métriques et alertes de sécurité ;
5. retirer `InternalServiceGuard` uniquement après migration complète et recette des routes concernées.
