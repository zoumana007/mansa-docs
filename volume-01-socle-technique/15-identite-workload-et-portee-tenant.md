# Identité workload attestée et portée tenant

## 1. Objet

Cette spécification définit la cible d’authentification service-à-service de Mansa. Elle remplace progressivement le secret partagé statique `INTERNAL_SERVICE_TOKEN` et interdit qu’une route sensible fasse confiance à un `organizationId` fourni arbitrairement par l’appelant.

La première utilisation prioritaire concerne le rapprochement financier, puis le ledger, les workers opérationnels et les adaptateurs partenaires.

## 2. Principes

1. Une identité workload représente un service ou worker déployé, jamais un utilisateur final.
2. L’organisation autorisée est portée par l’identité vérifiée, pas par un paramètre HTTP libre.
3. Les droits sont exprimés par scopes minimaux.
4. Les attestations sont courtes, rejouables le moins possible et identifiées par un `tokenId` unique.
5. Les routes restent fail-closed si l’identité ne peut pas être vérifiée.
6. L’identité vérifiée est propagée sous forme de contexte interne ; les contrôleurs ne décodent pas eux-mêmes les credentials.
7. Aucun secret, certificat privé ou jeton réel n’est stocké dans Git.

## 3. Contrat partagé

Le dépôt `mansa-platform` définit `packages/contracts/src/workload-identity.ts`.

Une identité contient au minimum :

- `version` ;
- `workloadId` ;
- `organizationId` ;
- `scopes` ;
- `issuedAt` ;
- `expiresAt` ;
- `tokenId`.

Durée maximale initiale : **15 minutes**. Une implémentation de production peut imposer une durée plus courte.

Scopes initiaux :

- `ledger:read` ;
- `ledger:write` ;
- `reconciliation:read` ;
- `reconciliation:write` ;
- `operations:read` ;
- `operations:write`.

Les scopes devront rester extensibles par ajout explicite et revue de sécurité.

## 4. Cible cryptographique

La cible de production est une identité de workload vérifiable via l’infrastructure de déploiement : OIDC signé ou identité mTLS/SPIFFE équivalente. Le vérificateur doit contrôler au minimum :

- signature ;
- émetteur autorisé ;
- audience Mansa attendue ;
- dates de validité ;
- identifiant unique ;
- workload autorisé ;
- organisation ;
- scopes.

Les clés publiques doivent être résolues depuis une source approuvée et mises en cache avec rotation. Une clé inconnue, expirée ou révoquée entraîne un refus.

## 5. Transition depuis le token statique

La migration se fait sans coupure :

### Phase A — contrat

- contrat `WorkloadIdentity` partagé ;
- validation stricte ;
- tests runtime ;
- aucune modification des routes existantes.

### Phase B — vérificateur et contexte

- service `WorkloadIdentityVerifier` ;
- middleware/guard qui attache un contexte attesté à la requête ;
- métriques de succès/échec ;
- token statique conservé uniquement comme fallback explicitement activé hors production.

### Phase C — rapprochement

- retrait de `organizationId` des paramètres publics internes de rapprochement ;
- organisation extraite du contexte workload ;
- scopes `reconciliation:read` et `reconciliation:write` exigés selon la route ;
- tests négatifs inter-tenant et scope insuffisant.

### Phase D — ledger et opérations

- même mécanisme appliqué au ledger et aux routes opérationnelles ;
- retrait du secret statique de la configuration production ;
- rotation/révocation documentées.

## 6. Contexte de requête

Après vérification, la requête doit exposer uniquement un contexte normalisé :

```text
workloadId
organizationId
scopes
tokenId
```

Le credential brut ne doit pas être transmis aux repositories ni journalisé.

Les repositories continuent d’exiger explicitement `organizationId`. La couche HTTP/application leur fournit l’organisation provenant du contexte attesté. Cela conserve la défense en profondeur déjà mise en place dans la persistance.

## 7. Règles de journalisation

Les journaux peuvent contenir :

- `workloadId` ;
- `organizationId` ;
- `tokenId` ;
- route ;
- décision d’autorisation ;
- code d’erreur ;
- corrélation.

Ils ne contiennent jamais le jeton signé, le secret partagé, une clé privée ou une preuve mTLS complète.

## 8. Rejeu et révocation

Pour les opérations sensibles d’écriture, le système doit pouvoir détecter un rejeu anormal de `tokenId` lorsque le fournisseur d’identité ne fournit pas déjà cette garantie. La stratégie exacte dépendra de l’infrastructure retenue.

Une procédure de révocation doit permettre de bloquer rapidement :

- un workload ;
- une clé de signature ;
- un issuer ;
- une organisation compromise ;
- un scope particulier si nécessaire.

## 9. Critères de recette

La tranche n’est considérée terminée que si :

1. une identité valide et non expirée est acceptée ;
2. une identité expirée, future, mal formée ou trop longue est refusée ;
3. un scope absent provoque `403` ;
4. une identité non authentifiée provoque `401` ou un échec fermé équivalent ;
5. une organisation A ne peut jamais accéder aux données de B ;
6. `organizationId` n’est plus accepté comme source d’autorité depuis la query/body ;
7. les tests PostgreSQL d’isolation existants restent verts ;
8. aucun credential brut n’apparaît dans les réponses ou logs ;
9. la rotation de clé peut être réalisée sans redéploiement applicatif complet si l’infrastructure choisie le permet ;
10. le fallback token statique est impossible en production après la phase finale.

## 10. État actuel

La **Phase A** est engagée dans `mansa-platform` avec le contrat partagé, la liste initiale de scopes, la validation de durée/format et les tests runtime. Les phases B à D restent à implémenter progressivement.

Cette étape est volontairement additive : elle ne modifie pas encore `InternalServiceGuard` et ne change donc pas le comportement des routes existantes avant que le vérificateur attesté et ses tests d’intégration soient prêts.
