# 17 — Agrégats de résultats métier du rapprochement

## Objet

Cette tranche complète l’observabilité locale du rapprochement financier avec des compteurs de résultats métier à cardinalité strictement bornée.

Elle prolonge :

- `15-contrat-export-metriques-rapprochement.md` ;
- `16-duree-imports-rapprochement.md`.

L’objectif est de répondre à des questions opérationnelles simples sans exposer de tenant, fournisseur, lot, transaction, fichier ou référence dans les métriques.

## État implémenté

Dans `mansa-platform` :

- `ReconciliationImportService` calcule le résumé métier des comparaisons préparées avant d’enregistrer le succès de l’import ;
- `ReconciliationOperationalMonitor` cumule :
  - les items `MATCHED` ;
  - les items `MISMATCHED` ;
  - les motifs de mismatch issus de l’énumération fermée du domaine ;
- `LowCardinalityReconciliationMetricsExporter` expose chaque agrégat avec un nom de métrique fixe, sans label dynamique.

Les échecs d’import n’ajoutent aucun résultat métier, car aucun batch réussi n’est alors considéré comme importé par le monitor.

## Métriques ajoutées

### Résultats principaux

- `mansa_reconciliation_matched_items_total`
- `mansa_reconciliation_mismatched_items_total`

Ces deux compteurs sont cumulés uniquement sur les imports terminés avec succès.

### Motifs de mismatch

Les motifs sont exportés par métriques distinctes et fixes :

- `mansa_reconciliation_mismatch_missing_internal_transaction_total`
- `mansa_reconciliation_mismatch_missing_provider_transaction_total`
- `mansa_reconciliation_mismatch_amount_total`
- `mansa_reconciliation_mismatch_currency_total`
- `mansa_reconciliation_mismatch_status_total`
- `mansa_reconciliation_mismatch_duplicate_provider_transaction_total`
- `mansa_reconciliation_mismatch_other_total`

Aucun `reason="..."` dynamique n’est utilisé. Ce choix rend la cardinalité indépendante du volume de données et empêche l’introduction accidentelle d’identifiants métier dans les dimensions de métriques.

## Invariants du monitor

Lorsqu’un résumé métier est fourni avec un succès d’import :

- `matched >= 0` ;
- `mismatched >= 0` ;
- les valeurs doivent être des entiers sûrs ;
- `matched + mismatched = itemCount` ;
- chaque compteur de motif doit être un entier sûr non négatif ;
- la somme des motifs ne peut pas dépasser `mismatched` ;
- un motif inconnu est refusé.

Le résumé métier est dérivé des comparaisons préparées par l’adaptateur, donc il ne dépend pas de chaînes arbitraires fournies à l’exporteur.

## Confidentialité et cardinalité

Les métriques ne contiennent aucune dimension pour :

- organisation ;
- tenant ;
- fournisseur ;
- utilisateur ;
- compte ;
- batch ;
- transaction ;
- fichier ;
- référence interne ;
- référence fournisseur.

Elles sont globales au processus courant et doivent être interprétées comme telles.

## Interprétation opérationnelle

Ces compteurs permettent notamment de calculer hors processus :

- taux de rapprochement exact = `matched / imported_items` ;
- taux de mismatch = `mismatched / imported_items` ;
- distribution agrégée des causes de mismatch ;
- évolution d’un motif dominant après changement de connecteur ou de format fournisseur.

Ils ne permettent volontairement pas de déterminer quel tenant ou quel fournisseur est à l’origine d’un écart.

## Tests

`apps/api-gateway/test/reconciliation-operational-monitor.test.mjs` couvre :

- agrégation `MATCHED` / `MISMATCHED` ;
- cumul entre plusieurs imports ;
- cumul des motifs ;
- rejet des résumés incohérents ;
- rejet d’un total de motifs supérieur au nombre de mismatches.

`apps/api-gateway/test/reconciliation-metrics-exporter.test.mjs` couvre :

- exposition des métriques fixes ;
- valeurs nulles pour les motifs non observés ;
- absence de labels et d’identifiants sensibles.

## Limites de cette tranche

Cette tranche ne fournit toujours pas :

- exposition HTTP/Prometheus ;
- backend de métriques externe ;
- histogrammes ou percentiles ;
- alertes ;
- SLI/SLO ;
- dimensions tenant ou fournisseur.

Les dimensions tenant/fournisseur ne doivent pas être ajoutées à ces séries process-locales sans revue explicite de confidentialité, de cardinalité et d’architecture d’observabilité.

## Prochaine tranche recommandée

Définir un point d’exposition interne des métriques avec authentification dédiée et format provider-neutral ou Prometheus/OpenMetrics, sans rendre ces agrégats globaux accessibles aux workloads métier ordinaires. Le choix du mécanisme d’accès doit garantir qu’un workload tenant-scopé ne peut pas observer l’activité agrégée des autres organisations.
