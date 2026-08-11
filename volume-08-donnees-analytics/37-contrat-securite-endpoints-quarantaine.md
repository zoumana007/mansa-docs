# 37 — Contrat de sécurité des endpoints internes de quarantaine

## Objet

Cette tranche verrouille par tests de contrat les contrôles de sécurité appliqués aux endpoints internes d’inventaire et de résumé des politiques de quarantaine de réconciliation.

L’objectif est d’empêcher qu’un refactor ultérieur retire silencieusement :

- l’authentification workload ;
- le contrôle de scope workload ;
- le scope obligatoire `reconciliation:read`.

Aucune donnée supplémentaire n’est exposée et aucun comportement métier de quarantaine n’est modifié.

## Endpoints concernés

```text
GET /v1/internal/reconciliation/quarantine-policies
GET /v1/internal/reconciliation/quarantine-policies/summary
```

Les deux routes restent strictement internes.

## Contrôles attendus au niveau du contrôleur

`ReconciliationQuarantinePolicyController` doit conserver les gardes de classe suivants, dans cet ordre :

1. `WorkloadIdentityGuard` ;
2. `WorkloadScopeGuard`.

Le premier garantit qu’une identité workload authentifiée est présente. Le second exige qu’une politique de scopes soit explicitement déclarée et vérifie les scopes de cette identité.

L’absence d’une politique de scope doit rester fail-closed.

## Contrôles attendus au niveau des méthodes

Les méthodes :

- `listPolicies()` ;
- `summarizePolicies()` ;

doivent toutes les deux porter :

```text
reconciliation:read
```

via `RequireWorkloadScopes`.

Un futur endpoint interne de réconciliation ne doit pas être considéré protégé uniquement parce que le contrôleur possède des gardes : le scope de méthode doit rester explicitement défini lorsque le garde l’exige.

## Tests ajoutés

Le fichier :

```text
apps/api-gateway/test/reconciliation-quarantine-policy.controller.test.mjs
```

vérifie désormais en plus des tests fonctionnels existants :

- que la métadonnée Nest des gardes du contrôleur contient exactement `WorkloadIdentityGuard` et `WorkloadScopeGuard` ;
- que `summarizePolicies()` exige exactement `reconciliation:read` ;
- que `listPolicies()` exige également exactement `reconciliation:read`.

Les tests lisent directement les métadonnées générées par les décorateurs Nest. Ils ne simulent donc pas uniquement le comportement d’un mock : ils verrouillent la configuration déclarative réellement utilisée par le framework.

## Pourquoi tester les métadonnées

Un test de réponse HTTP ou de méthode seule pourrait continuer à passer même si un décorateur était supprimé par erreur dans le code source.

Le contrat de métadonnées apporte une défense supplémentaire contre plusieurs régressions :

- suppression accidentelle de `UseGuards` ;
- remplacement incomplet des gardes ;
- suppression de `RequireWorkloadScopes` ;
- changement involontaire du scope requis ;
- divergence entre l’inventaire détaillé et le résumé agrégé.

## Garanties conservées

Cette tranche ne change pas les garanties précédentes :

- aucune exposition de secret ;
- aucun payload fournisseur ;
- aucun stockage brut activé par l’endpoint ;
- aucune autorisation implicite de persistance ;
- politiques non approuvées toujours refusées par les chemins métier concernés ;
- cardinalité bornée du résumé ;
- inventaire réservé aux workloads internes autorisés.

## Validation attendue

Après compilation de l’API Gateway, les tests Node du contrôleur doivent pouvoir importer les classes produites dans `dist/` et vérifier les métadonnées des décorateurs.

La validation complète du dépôt doit continuer à couvrir :

- formatage ;
- lint ;
- TypeScript/build ;
- tests unitaires et d’intégration pertinents ;
- contrôles CI existants.

## Cohérence documentaire

Cette tranche complète directement :

- `36-resume-operationnel-politiques-quarantaine.md` ;
- les documents précédents sur le registre de politiques, la garde de persistance et l’inventaire interne.

Elle ferme le prochain écart explicitement recommandé par le document 36 : la sécurité déclarative des endpoints de supervision est désormais couverte par un test de contrat dédié.

## Prochaine tranche recommandée

Étendre la supervision interne avec un contrat de santé/configuration non sensible permettant de distinguer une configuration vide d’une configuration présente mais non exploitable, sans exposer d’identifiant fournisseur, de rôle, de durée de rétention ou de payload brut.
