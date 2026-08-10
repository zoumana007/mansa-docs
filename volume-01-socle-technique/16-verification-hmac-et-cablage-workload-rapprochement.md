# 16 — Vérification HMAC et câblage workload du rapprochement

## Objet

Ce document décrit la première implémentation cryptographique exploitable de l’identité workload et son câblage au moteur de rapprochement financier.

## Décision

Le socle supporte désormais un credential JWT signé en `HS256` pour les communications internes. Cette implémentation est volontairement derrière l’interface `WorkloadIdentityVerifier` afin de pouvoir être remplacée ultérieurement par OIDC/JWKS, mTLS/SPIFFE ou un mécanisme équivalent sans modifier les contrôleurs métier.

Le secret symétrique n’est jamais stocké dans Git. Il doit être fourni au runtime via :

- `WORKLOAD_IDENTITY_HMAC_SECRET` — minimum 32 octets ;
- `WORKLOAD_IDENTITY_ISSUER` — issuer attendu ;
- `WORKLOAD_IDENTITY_AUDIENCE` — audience attendue.

Le verifier refuse :

- tout token ne contenant pas exactement trois segments JWT ;
- tout algorithme différent de `HS256` ;
- toute signature invalide ;
- tout issuer inattendu ;
- toute audience inattendue ;
- toute identité rejetée ensuite par `validateWorkloadIdentity` (version, UUID, scopes, dates, expiration, durée maximale de 15 minutes, etc.).

La comparaison de signature utilise `timingSafeEqual`.

## Câblage du rapprochement

Les routes `internal/reconciliation/v1` n’utilisent plus `InternalServiceGuard` ni un `organizationId` fourni librement par query string.

Elles appliquent désormais :

1. `WorkloadIdentityGuard` ;
2. `WorkloadScopeGuard` ;
3. `RequireWorkloadScopes(...)` selon l’opération.

Scopes :

- lectures : `reconciliation:read` ;
- résolution : `reconciliation:write`.

Le `organizationId` transmis au repository provient exclusivement de `request.workloadIdentity.organizationId`.

Pour une résolution manuelle orchestrée par un workload, l’identité d’audit n’est plus fournie par le body. Le backend impose :

- `actorId = workloadIdentity.workloadId` ;
- `actorType = WORKLOAD`.

Cela évite qu’un appelant interne forge son tenant ou son identité d’audit.

## Compatibilité API

Le paramètre query `organizationId` est supprimé des routes de rapprochement. Les clients internes doivent migrer vers un bearer workload signé et recevoir les scopes adéquats.

Les filtres fonctionnels existants, la pagination keyset, les DTO publics, l’idempotence et l’isolation PostgreSQL restent inchangés.

## Sécurité opérationnelle

Le mode HMAC implique un secret partagé et doit être traité comme un palier de transition sûr mais non comme l’architecture finale idéale pour un grand nombre de services. En production distribuée, privilégier à terme des identités asymétriques avec rotation indépendante, JWKS ou SPIFFE/mTLS.

Le secret doit être injecté par un gestionnaire de secrets, avoir une entropie suffisante, être rotatable, ne jamais être journalisé et ne jamais apparaître dans un fichier `.env` commité.

## Tests attendus

La couverture doit vérifier au minimum :

- dérivation du tenant depuis le contexte workload ;
- rejet d’un contexte absent ;
- absence de fuite de `organizationId` dans les DTO publics ;
- propagation des filtres ;
- résolution avec acteur imposé par le workload ;
- scopes read/write via les guards dédiés ;
- signature, issuer et audience du verifier ;
- expiration et durée maximale via le validateur partagé.

## Suite

Les prochaines tranches prioritaires sont :

1. tests unitaires dédiés du verifier HMAC et intégration NestJS complète ;
2. rotation de clés / stratégie multi-clés ou migration vers JWKS/mTLS ;
3. câblage de la même identité workload aux autres routes internes encore protégées par le mécanisme historique ;
4. métriques et alertes d’authentification refusée, sans exposer les credentials ;
5. premiers adaptateurs partenaires de rapprochement réels après validation de sécurité.
