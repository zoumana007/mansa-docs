# 15 — Contrat d’export de métriques du rapprochement

## Objet

Cette tranche introduit un contrat d’export de métriques indépendant du fournisseur pour le moteur de rapprochement financier.

L’objectif est de rendre les métriques consommables ultérieurement par Prometheus, OpenTelemetry, CloudWatch, Datadog ou un autre backend sans faire dépendre le domaine de rapprochement d’un produit d’observabilité particulier.

## État implémenté

`mansa-platform/apps/api-gateway/src/reconciliation/reconciliation-metrics-exporter.ts` définit :

- `ReconciliationMetricsExporter`, contrat générique d’export ;
- `ReconciliationMetricSample`, représentation minimale d’un échantillon ;
- `RECONCILIATION_METRICS_EXPORTER`, token d’injection NestJS ;
- `LowCardinalityReconciliationMetricsExporter`, implémentation locale à faible cardinalité.

Le module `ReconciliationModule` enregistre cette implémentation derrière le token abstrait afin qu’un exporter concret puisse être remplacé ultérieurement sans modifier le service d’import.

## Métriques exposées dans cette tranche

Le premier jeu volontairement réduit expose seulement :

- `mansa_reconciliation_imports_started_total` ;
- `mansa_reconciliation_imports_succeeded_total` ;
- `mansa_reconciliation_imports_failed_total` ;
- `mansa_reconciliation_imported_items_total`.

Ces métriques sont des compteurs process-local dérivés du `ReconciliationOperationalMonitor` déjà présent.

## Politique de cardinalité

Cette tranche n’ajoute aucun label dynamique.

Sont notamment interdits comme dimensions de métriques :

- identifiant de tenant ou d’organisation ;
- identifiant de transaction ;
- identifiant de lot ;
- référence client ;
- référence fournisseur ;
- nom de fichier ;
- empreinte de fichier ou de ligne ;
- identifiant utilisateur ;
- valeur monétaire individuelle.

Cette contrainte évite les séries à cardinalité non bornée, les risques de fuite de données et l’explosion des coûts de stockage des métriques.

## Données sensibles

Le contrat ne contient que :

- nom technique de métrique ;
- type de métrique ;
- valeur numérique agrégée ;
- unité.

Aucun payload de rapprochement ni secret n’est exporté.

## Tests

`apps/api-gateway/test/reconciliation-metrics-exporter.test.mjs` vérifie :

- la projection correcte des compteurs du monitor ;
- le cumul des imports et items ;
- l’absence de labels ;
- l’absence d’identifiants organisation, fournisseur ou transaction dans les samples.

Le test suit le mécanisme existant du dépôt : compilation TypeScript de l’API Gateway puis exécution des fichiers `test/*.test.mjs`.

## Limites de cette tranche

Cette implémentation ne constitue pas encore un endpoint `/metrics` ni une intégration Prometheus/OpenTelemetry de production.

Il reste à ajouter par lots distincts :

1. mesure de durée des imports ;
2. agrégats par résultat métier à cardinalité bornée ;
3. backlog non résolu et âge du plus ancien écart ;
4. résolutions opérateur ;
5. exporter concret choisi pour l’environnement cible ;
6. SLI/SLO ;
7. règles d’alerte ;
8. dashboards ;
9. corrélation avec traces et logs.

## Prochaine tranche recommandée

Étendre le monitor avec une mesure de durée d’import monotone et tester les cas succès/échec, puis publier cette durée derrière le même contrat d’export sans introduire de label à forte cardinalité.
