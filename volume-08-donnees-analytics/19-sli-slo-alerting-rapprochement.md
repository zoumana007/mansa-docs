# 19 — SLI, SLO et règles d’alerte du rapprochement

## Objectif

Cette tranche transforme les métriques bornées du moteur de rapprochement en indicateurs de fiabilité exploitables, sans coupler le domaine métier à Prometheus, Grafana, Datadog, CloudWatch ou un autre fournisseur.

Le runtime doit pouvoir évaluer localement une politique SLI/SLO déterministe à partir des métriques déjà exposées par `LowCardinalityReconciliationMetricsExporter`.

## SLI retenus

Trois indicateurs sont retenus dans cette première version.

### Taux d’échec des imports

```text
importFailureRate = importsFailed / (importsSucceeded + importsFailed)
```

Lorsque aucun import n’est terminé, la valeur est `null` et ne doit pas être interprétée comme `0 %`.

### Taux d’écart métier

```text
mismatchRate = mismatchedItems / (matchedItems + mismatchedItems)
```

Lorsque aucun élément n’a été comparé, la valeur est `null`.

### Durée du dernier import terminé

Source :

```text
mansa_reconciliation_last_import_duration_ms
```

Cette métrique reste process-local. Elle n’est pas encore un percentile et ne doit pas être présentée comme un p95 ou p99.

## SLO par défaut

La politique initiale utilise les objectifs suivants :

```text
maximumImportFailureRate = 1 %
maximumMismatchRate = 0,5 %
maximumLastImportDurationMs = 30 000 ms
```

Ces valeurs sont des valeurs techniques initiales et doivent rester configurables dans les couches d’exploitation futures. Elles ne constituent pas un engagement contractuel client.

## États

L’évaluation retourne un état fermé :

```text
NO_DATA
HEALTHY
WARNING
CRITICAL
```

### NO_DATA

Aucun import terminé, aucun élément comparé et aucune durée de dernier import n’est disponible.

`NO_DATA` n’est pas assimilé à `HEALTHY`.

### HEALTHY

Tous les SLI disponibles sont inférieurs ou égaux aux seuils définis.

### WARNING

Au moins un seuil est dépassé sans atteindre le niveau critique.

### CRITICAL

Au moins un SLI atteint ou dépasse deux fois son seuil autorisé.

Cette règle simple permet une première classification déterministe. Un futur moteur d’alerting pourra utiliser des fenêtres temporelles, burn rates et politiques multi-fenêtres sans modifier les contrats métier.

## Contrat des violations

Chaque violation doit contenir uniquement :

- l’indicateur fermé ;
- la sévérité ;
- la valeur observée ;
- le seuil appliqué.

Aucun identifiant de tenant, transaction, batch, fichier, fournisseur ou utilisateur ne doit être ajouté à cette structure.

Indicateurs autorisés :

```text
IMPORT_FAILURE_RATE
MISMATCH_RATE
LAST_IMPORT_DURATION_MS
```

## Sécurité et confidentialité

La politique SLO travaille uniquement sur les métriques agrégées et bornées déjà autorisées.

Elle ne doit pas :

- relire les transactions depuis Prisma ;
- charger des données client ;
- recalculer les écarts à partir des lignes de rapprochement ;
- introduire des labels dynamiques ;
- exposer les identifiants internes ;
- envoyer directement des notifications externes ;
- contenir de credential de monitoring.

## Implémentation runtime

Le runtime fournit `ReconciliationSloPolicy`.

Cette classe :

- consomme le tableau de `ReconciliationMetricSample` ;
- dérive les SLI ;
- applique les seuils ;
- retourne un résultat immuable ;
- accepte des seuils explicites pour permettre les tests et les futures configurations ;
- conserve des valeurs par défaut immuables ;
- n’a aucune dépendance vers un fournisseur de monitoring.

## Tests attendus

La tranche doit couvrir au minimum :

1. `NO_DATA` lorsqu’aucune activité terminée n’existe ;
2. `HEALTHY` lorsque tous les indicateurs respectent les objectifs ;
3. `WARNING` lors d’un dépassement modéré ;
4. `CRITICAL` lorsqu’un seuil est au moins doublé ;
5. plusieurs violations simultanées ;
6. utilisation de seuils personnalisés sans mutation des valeurs par défaut.

## Ce que cette tranche ne fait pas encore

Cette tranche ne fournit pas encore :

- stockage de séries temporelles ;
- calcul p95/p99 ;
- fenêtres glissantes ;
- error budget mensuel ;
- burn-rate multi-fenêtres ;
- endpoint dédié d’état SLO ;
- envoi PagerDuty, Slack, e-mail ou SMS ;
- dashboards ;
- agrégation multi-instance ;
- configuration dynamique en base.

## Prochaine tranche recommandée

La suite logique est d’introduire un **adaptateur d’alerting provider-neutral** consommant `ReconciliationSloEvaluation`, avec déduplication, fenêtre de refroidissement et transitions d’état (`HEALTHY → WARNING → CRITICAL → RECOVERED`) avant de raccorder un backend réel de supervision.
