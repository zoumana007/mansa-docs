# 20 — Alerting provider-neutral du rapprochement

## Objectif

Cette tranche complète la politique SLI/SLO du rapprochement avec une couche de décision d’alerte indépendante de tout fournisseur externe.

Le domaine ne doit appeler directement ni PagerDuty, ni Slack, ni e-mail, ni SMS, ni un service de monitoring propriétaire. Il doit uniquement décider si un événement mérite d’être notifié, selon l’état SLO courant, l’état précédent et une fenêtre de refroidissement.

## Entrée

La politique consomme exclusivement un `ReconciliationSloEvaluation` déjà calculé.

États possibles :

```text
NO_DATA
HEALTHY
WARNING
CRITICAL
```

La politique ne relit aucune transaction, aucun batch, aucun utilisateur et aucune donnée client.

## Événements bornés

Les événements sortants sont fermés :

```text
WARNING
CRITICAL
RECOVERED
REMINDER
```

### WARNING

Émis lors d’une transition vers `WARNING`.

### CRITICAL

Émis lors d’une transition vers `CRITICAL`, y compris lorsqu’un état `WARNING` devient `CRITICAL` avant la fin du cooldown.

### RECOVERED

Émis lorsqu’un état précédemment `WARNING` ou `CRITICAL` redevient `HEALTHY`.

### REMINDER

Émis uniquement si un même état dégradé persiste au-delà de la fenêtre de refroidissement.

## Déduplication

Deux évaluations consécutives avec le même état dégradé ne doivent pas déclencher deux notifications immédiates.

Exemple :

```text
12:00 WARNING -> notification WARNING
12:01 WARNING -> aucune notification
12:05 WARNING -> aucune notification
12:15 WARNING -> REMINDER si cooldown = 15 minutes
```

La déduplication est volontairement basée sur un état fermé et non sur des identifiants métier afin d’éviter une explosion de cardinalité.

## Transitions

Transitions significatives :

```text
HEALTHY -> WARNING  => WARNING
HEALTHY -> CRITICAL => CRITICAL
WARNING -> CRITICAL => CRITICAL immédiatement
CRITICAL -> WARNING => WARNING immédiatement
WARNING -> HEALTHY  => RECOVERED
CRITICAL -> HEALTHY => RECOVERED
```

`NO_DATA` ne déclenche aucune notification.

Un état `HEALTHY` stable reste silencieux.

## Cooldown

Valeur technique initiale :

```text
15 minutes
```

Le cooldown doit être configurable par appel de la politique et ne doit pas être codé dans un adaptateur fournisseur.

Il s’applique uniquement aux rappels d’un état dégradé inchangé.

Une transition de sévérité ne doit pas attendre la fin du cooldown.

## Contrat de décision

Chaque décision expose uniquement :

- `shouldNotify` ;
- `event` ;
- `reason` ;
- `previousStatus` ;
- `currentStatus` ;
- `evaluatedAt` ;
- `nextEligibleReminderAt`.

Raisons autorisées :

```text
STATE_CHANGE
COOLDOWN_ELAPSED
COOLDOWN_ACTIVE
NO_DATA
HEALTHY_STEADY
```

Aucun identifiant de client, fournisseur, transaction, batch ou fichier ne doit apparaître dans ce contrat.

## État runtime

La première implémentation maintient un état **process-local** :

- dernier statut ;
- horodatage de la dernière notification.

Cette limite est assumée pour la première tranche.

En environnement multi-réplique, un futur adaptateur partagé devra déplacer cet état vers une couche distribuée sans changer le contrat de décision. Redis ou une autre solution pourra être utilisée plus tard, mais aucune dépendance de ce type n’est introduite dans cette tranche.

## Sécurité

La politique :

- ne contient aucun secret ;
- ne possède aucune URL de webhook ;
- ne connaît aucun token PagerDuty/Slack ;
- ne transmet aucune donnée métier ;
- ne persiste aucune donnée sensible ;
- n’effectue aucun appel réseau ;
- produit uniquement un objet de décision borné.

Les credentials d’un futur adaptateur devront rester hors Git et être fournis par la configuration sécurisée de l’environnement.

## Tests attendus

La tranche doit couvrir :

1. `NO_DATA` silencieux ;
2. première transition vers `WARNING` ;
3. déduplication d’un `WARNING` répété ;
4. rappel après expiration du cooldown ;
5. escalade `WARNING -> CRITICAL` immédiate ;
6. récupération `CRITICAL -> HEALTHY` ;
7. état `HEALTHY` stable silencieux ;
8. validation du cooldown ;
9. remise à zéro explicite pour tests et bootstrap contrôlé.

## Implémentation runtime

Le runtime fournit :

```text
ReconciliationAlertingPolicy
```

La classe est injectable par NestJS et exportée par `ReconciliationModule` afin qu’un futur composant de monitoring puisse la consommer sans dupliquer les règles.

## Hors périmètre de cette tranche

Cette tranche ne fournit pas encore :

- persistance distribuée de l’état ;
- routage par équipe ;
- Slack ;
- PagerDuty ;
- e-mail ;
- SMS ;
- escalade d’astreinte ;
- fenêtres multi-burn-rate ;
- error budgets mensuels ;
- dashboard ;
- endpoint public ;
- stockage historique des alertes.

## Prochaine tranche recommandée

La suite logique est de créer un **orchestrateur de monitoring du rapprochement** qui :

1. lit les métriques bornées ;
2. calcule le SLO via `ReconciliationSloPolicy` ;
3. demande une décision à `ReconciliationAlertingPolicy` ;
4. transmet uniquement les décisions `shouldNotify = true` à un port `ReconciliationAlertSink` ;
5. utilise d’abord un sink de test/mémoire avant toute intégration externe.

Cette séparation permettra de tester toute la chaîne de supervision sans introduire de dépendance fournisseur ni de secret.
