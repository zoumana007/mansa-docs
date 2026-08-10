# 16 — Durée monotone des imports de rapprochement

## Objet

Cette tranche ajoute une mesure de durée des imports de rapprochement financier sans dépendre de l’horloge murale et sans introduire de dimensions à forte cardinalité.

Elle prolonge le contrat d’export de métriques défini dans `15-contrat-export-metriques-rapprochement.md`.

## État implémenté

Dans `mansa-platform` :

- `ReconciliationImportService` démarre un chronomètre monotone avec `performance.now()` au début de chaque import ;
- la durée est enregistrée aussi bien en cas de succès qu’en cas d’échec ;
- `ReconciliationOperationalMonitor` conserve :
  - le cumul des durées des imports terminés ;
  - la durée du dernier import terminé ;
- `LowCardinalityReconciliationMetricsExporter` publie ces valeurs derrière le contrat existant.

L’horloge murale (`Date`) reste utilisée uniquement pour les timestamps opérationnels `lastImport*At`. La mesure de durée repose sur une source monotone afin d’éviter les effets d’un changement d’heure système pendant l’opération.

## Nouvelles métriques

### `mansa_reconciliation_import_duration_ms_total`

- type : `COUNTER` ;
- unité : `milliseconds` ;
- valeur : somme des durées de tous les imports terminés dans le processus courant ;
- inclut succès et échecs.

Cette métrique permet notamment de calculer une durée moyenne process-local en la divisant par :

`mansa_reconciliation_imports_succeeded_total + mansa_reconciliation_imports_failed_total`.

### `mansa_reconciliation_last_import_duration_ms`

- type : `GAUGE` ;
- unité : `milliseconds` ;
- valeur : durée du dernier import terminé, qu’il ait réussi ou échoué ;
- absente tant qu’aucun import n’a terminé.

## Politique de confidentialité et cardinalité

Aucun label dynamique n’est ajouté.

La durée n’est pas ventilée par :

- tenant ;
- organisation ;
- fournisseur ;
- lot ;
- transaction ;
- fichier ;
- utilisateur ;
- référence interne ou externe.

La mesure reste donc bornée et ne révèle pas d’identifiant métier dans le backend de métriques.

## Validation des données

Le monitor refuse toute durée :

- négative ;
- `NaN` ;
- infinie.

Les durées valides peuvent être décimales car `performance.now()` fournit une résolution sous-milliseconde selon l’environnement.

## Tests

`apps/api-gateway/test/reconciliation-operational-monitor.test.mjs` couvre :

- durée d’un succès ;
- durée d’un échec ;
- cumul des durées entre succès et échecs ;
- rejet des durées invalides.

`apps/api-gateway/test/reconciliation-metrics-exporter.test.mjs` couvre :

- exposition du compteur de durée totale ;
- exposition du gauge de dernière durée ;
- absence du gauge avant le premier import terminé ;
- maintien de l’absence de labels et d’identifiants sensibles.

## Limites de cette tranche

Cette tranche ne fournit pas encore :

- histogramme de latence ;
- percentiles p50/p95/p99 ;
- buckets Prometheus ;
- ventilation par résultat métier ;
- SLI/SLO de latence ;
- règle d’alerte sur import lent.

Ces éléments doivent être ajoutés seulement après choix du backend d’observabilité et définition de seuils opérationnels réalistes.

## Prochaine tranche recommandée

Ajouter des agrégats de résultat métier à cardinalité bornée pour les items rapprochés : `MATCHED`, `MISMATCH`, `MISSING_INTERNAL`, `MISSING_PROVIDER` et autres statuts déjà définis par le domaine, sans exposer d’identifiants de transaction ou de fournisseur.
