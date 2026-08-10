# 18 — Exposition interne protégée des métriques de rapprochement

## Objectif

Le moteur de rapprochement dispose désormais d’un monitoring process-local et d’un export de métriques à faible cardinalité. Ce document définit la première surface HTTP interne permettant à une brique d’observabilité autorisée de lire ces métriques sans exposer les données métier, les identifiants de tenant, de transaction, de lot, de fichier ou de fournisseur.

## Endpoint

Version initiale :

```text
GET /v1/internal/metrics/reconciliation
```

La réponse contient :

```json
{
  "data": [
    {
      "name": "mansa_reconciliation_imports_succeeded_total",
      "kind": "COUNTER",
      "value": 42,
      "unit": "count"
    }
  ]
}
```

Les noms de métriques sont fermés et définis par l’exporter. Aucun label dynamique n’est accepté dans cette tranche.

## Authentification et autorisation

L’endpoint utilise l’identité workload interne déjà attestée par le socle HMAC.

Un scope dédié est ajouté :

```text
reconciliation:metrics:read
```

Ce scope doit être réservé aux workloads d’observabilité de plateforme et ne doit pas être distribué aux workloads métier tenant-scopés ordinaires.

Le fait qu’une identité possède `reconciliation:read`, `operations:read` ou des droits d’administration métier ne lui donne pas implicitement accès à cette surface.

La politique d’émission de credentials doit donc respecter les règles suivantes :

- attribution explicite du scope `reconciliation:metrics:read` ;
- durée de vie courte des credentials, conformément au contrat workload existant ;
- workload d’observabilité dédié ;
- aucun partage de credential avec une application Client, Commerce, Agent ou partenaire ;
- rotation et révocation possibles ;
- audit de la politique d’émission hors de la surface de métriques elle-même.

## Isolation et confidentialité

Les métriques exportées sont globales au processus et peuvent agréger l’activité de plusieurs organisations. Pour cette raison, elles ne doivent pas être exposées à un workload tenant-scopé simplement parce qu’il possède un tenant valide.

La protection repose sur un scope spécifique d’observabilité de rapprochement. L’endpoint ne renvoie jamais :

- `organizationId` ;
- `providerId` ;
- identifiant de batch ;
- identifiant de transaction ;
- nom de fichier ;
- référence interne ou partenaire ;
- données de paiement ;
- PII ;
- labels à cardinalité non bornée.

## Métriques exposées

La surface reprend le contrat défini par `LowCardinalityReconciliationMetricsExporter`, notamment :

- imports démarrés ;
- imports réussis ;
- imports échoués ;
- éléments importés ;
- éléments rapprochés ;
- éléments en écart ;
- motifs d’écart issus de l’énumération fermée ;
- durée cumulée des imports terminés ;
- durée du dernier import terminé lorsqu’elle existe.

## Propriétés de sécurité

- lecture seule ;
- aucune mutation métier ;
- aucune donnée Prisma brute ;
- aucune pagination nécessaire dans cette tranche ;
- aucune dimension dynamique ;
- aucune négociation de fournisseur ;
- aucun secret dans la réponse ;
- refus automatique des identities sans le scope dédié via `WorkloadScopeGuard`.

## Tests attendus

Le lot doit couvrir au minimum :

1. le nouveau scope fait partie des scopes workload reconnus ;
2. une identité possédant `reconciliation:metrics:read` passe le contrôle de scope ;
3. une identité possédant seulement `reconciliation:read` est refusée ;
4. l’endpoint renvoie uniquement le tableau produit par l’exporter ;
5. l’endpoint n’introduit aucun label tenant/provider/batch ;
6. les contrôles d’identité workload existants restent applicables.

## Limites de cette tranche

Cette tranche ne définit pas encore :

- un endpoint Prometheus text/plain ;
- une fédération multi-instance ;
- une persistance des séries temporelles ;
- les SLI/SLO ;
- les seuils d’alerte ;
- les dashboards ;
- les adaptateurs partenaires réels.

Ces sujets doivent venir dans des lots séparés afin de conserver une surface petite, contrôlable et auditable.

## Prochaine tranche

La prochaine étape recommandée est de définir les SLI/SLO et les règles d’alerte du rapprochement à partir des métriques bornées déjà disponibles, puis de raccorder un backend de monitoring réel derrière un adaptateur sans changer les contrats métier.
