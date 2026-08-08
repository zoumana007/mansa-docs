# Implémentation des lectures Ledger dans l’API Gateway

## 1. Portée du lot

Ce lot branche les premières lectures persistées du grand livre dans `mansa-platform/apps/api-gateway` sans ouvrir les commandes d’écriture. Il couvre les trois opérations de consultation suivantes :

- lecture d’un compte ;
- lecture de la projection de solde ;
- pagination des écritures d’un compte.

Les opérations de publication et de compensation restent hors de ce lot tant que l’atomicité PostgreSQL, l’idempotence transactionnelle, l’outbox et les contrôles de concurrence ne sont pas implémentés ensemble.

## 2. Composants backend

L’API Gateway contient désormais :

- `src/prisma.service.ts` : client Prisma partagé avec déconnexion propre à l’arrêt ;
- `src/ledger-read.service.ts` : mapping des modèles Prisma vers des réponses sérialisables ;
- `src/ledger.controller.ts` : routes HTTP de lecture ;
- `src/ledger.module.ts` : assemblage NestJS du domaine ;
- `src/internal-service.guard.ts` : protection transitoire service-à-service, fail-closed.

Le module est importé par `AppModule`.

## 3. Routes exposées

Le préfixe global actuel de l’API Gateway étant `api` et la version URI `1`, les chemins HTTP effectifs sont :

- `GET /api/v1/internal/ledger/accounts/:accountId` ;
- `GET /api/v1/internal/ledger/accounts/:accountId/balance` ;
- `GET /api/v1/internal/ledger/accounts/:accountId/entries`.

Le contrat métier reste formulé sans le préfixe de déploiement dans `10-contrat-api-ledger.md`. Les reverse proxies ou gateways externes ne doivent pas publier ces routes sur Internet.

## 4. Pagination des écritures

La liste des écritures applique l’ordre stable :

1. `postedAt ASC` ;
2. `id ASC`.

Le curseur encode la paire `(postedAt, entryId)` et la requête suivante utilise une condition keyset stricte. La limite par défaut est `50`, avec un maximum de `100`. Le service charge `limit + 1` éléments pour déterminer l’existence d’une page suivante sans requête de comptage.

Les paramètres `from` et `to` acceptent des dates ISO-8601. Une plage inversée est rejetée.

## 5. Sérialisation monétaire

Les colonnes Prisma `BigInt` ne sont jamais renvoyées directement au contrôleur car JSON ne sait pas sérialiser nativement `bigint`. Les champs suivants sont donc exposés sous forme de chaînes décimales :

- `availableMinor` ;
- `pendingMinor` ;
- `projectionSequence` ;
- `amountMinor`.

Aucun calcul en nombre flottant n’est introduit.

## 6. Protection des routes internes

Les trois routes utilisent `InternalServiceGuard`. Le guard exige l’en-tête :

`x-mansa-internal-token`

et compare sa valeur avec `INTERNAL_SERVICE_TOKEN` en temps constant lorsque les tailles correspondent.

Le guard est volontairement fail-closed : si `INTERNAL_SERVICE_TOKEN` est absent ou contient moins de 32 caractères, la route n’est pas rendue utilisable. Aucune valeur de production n’est versionnée ; `.env.example` ne contient qu’un placeholder.

Cette protection est transitoire. À terme, l’identité de workload signée ou le mTLS est préférable pour les appels inter-services, avec RBAC/ABAC complémentaire selon le domaine appelant.

## 7. Gestion d’erreurs

Le contrôleur rejette notamment :

- UUID de compte invalide ;
- compte ou projection introuvable ;
- date invalide ;
- plage `from > to` ;
- limite hors de `1..100` ;
- curseur invalide ;
- appel interne non authentifié.

Les erreurs continuent de traverser le filtre HTTP et le mécanisme de corrélation communs de l’API Gateway.

## 8. Limites et prochain lot

Ce lot ne signifie pas que le grand livre est prêt pour une mise en production. Restent notamment :

- génération et validation de la migration Prisma dans un PostgreSQL de test ;
- publication atomique d’une transaction ;
- idempotence avec empreinte de requête ;
- compensation ;
- mise à jour atomique des projections ;
- outbox transactionnelle et worker de publication ;
- réconciliation ;
- audit métier ;
- tests d’intégration PostgreSQL et concurrence ;
- métriques et alertes ;
- remplacement du jeton partagé par une identité inter-services plus robuste avant exposition à grande échelle.

Le prochain lot backend doit prioriser la commande atomique de publication et ses tests d’intégration, sans exposer de demi-transaction comptable.
