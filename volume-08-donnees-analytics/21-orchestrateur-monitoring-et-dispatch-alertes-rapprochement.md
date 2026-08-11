# 21 — Orchestrateur de monitoring et dispatch d’alertes du rapprochement

## Objectif

Cette tranche relie les composants de supervision du rapprochement déjà présents sans introduire de fournisseur externe.

Le cycle complet devient :

```text
ReconciliationOperationalMonitor
-> métriques bornées
-> ReconciliationSloPolicy
-> ReconciliationAlertingPolicy
-> ReconciliationAlertDispatcher
-> ReconciliationAlertSink
```

La chaîne doit rester indépendante de PagerDuty, Slack, e-mail, SMS ou d’un outil propriétaire.

## Principes

Le domaine de rapprochement ne doit connaître :

- aucune URL de webhook ;
- aucun token fournisseur ;
- aucun secret de notification ;
- aucun identifiant de client ;
- aucun identifiant de transaction ;
- aucun identifiant de fichier ou de batch dans les métriques ou les alertes.

Les intégrations externes doivent être branchées exclusivement derrière un port provider-neutral.

## ReconciliationAlertSink

Le port d’envoi est :

```text
ReconciliationAlertSink
```

Il expose une seule responsabilité : envoyer un `ReconciliationAlertPayload`.

Le payload est volontairement borné à :

- événement d’alerte ;
- statut SLO ;
- horodatage d’évaluation ;
- statut précédent ;
- breaches SLO ;
- snapshot SLI borné.

Aucune donnée métier brute ne doit être ajoutée à ce contrat.

## Sink par défaut

Le runtime fournit :

```text
NoopReconciliationAlertSink
```

Ce sink ne fait aucun appel réseau et ne possède aucun secret.

Il permet :

- de compiler et tester toute la chaîne ;
- de garder l’environnement local indépendant d’un fournisseur ;
- de remplacer ultérieurement le sink par une intégration réelle via injection de dépendance.

Le token d’injection est :

```text
RECONCILIATION_ALERT_SINK
```

## ReconciliationAlertDispatcher

Le dispatcher :

1. reçoit un `ReconciliationSloEvaluation` ;
2. demande une décision à `ReconciliationAlertingPolicy` ;
3. ne fait rien lorsque `shouldNotify = false` ;
4. construit un payload borné lorsque `shouldNotify = true` ;
5. appelle le sink ;
6. retourne la décision et l’état `delivered`.

Une erreur du sink ne doit pas être transformée en succès silencieux.

Si l’envoi échoue, la promesse doit échouer afin qu’un worker ou scheduler puisse appliquer sa propre stratégie de retry, DLQ ou incident.

## ReconciliationMonitoringOrchestrator

L’orchestrateur exécute un cycle complet :

1. prend un snapshot de `ReconciliationOperationalMonitor` ;
2. exporte les métriques via `RECONCILIATION_METRICS_EXPORTER` ;
3. calcule le SLO via `ReconciliationSloPolicy` ;
4. transmet l’évaluation au dispatcher ;
5. retourne un résultat borné contenant : métriques, évaluation SLO et résultat d’alerting.

Il ne planifie pas lui-même les cycles.

Il ne doit contenir :

- ni cron ;
- ni timer permanent ;
- ni retry réseau ;
- ni logique PagerDuty/Slack ;
- ni persistance distribuée.

Cette séparation permet à un futur worker, scheduler Kubernetes, queue worker ou service d’exploitation de déclencher `runCycle()` sans déplacer la logique métier de supervision.

## Configuration par cycle

Un cycle peut recevoir :

- seuils SLO ;
- cooldown d’alerting ;
- timestamp d’évaluation contrôlé.

Les valeurs par défaut restent celles définies par les politiques existantes.

Les tests peuvent injecter un timestamp déterministe afin d’éviter les flakiness temporelles.

## Contrat de résultat

Un cycle retourne :

```text
{
  metrics,
  evaluation,
  alerting
}
```

`metrics` reste sans labels à cardinalité élevée.

`evaluation` expose uniquement les SLI et breaches bornés.

`alerting` indique la décision et si un sink a réellement été appelé.

## Sécurité

Les règles suivantes sont obligatoires :

- aucun secret dans Git ;
- aucun endpoint de notification codé en dur ;
- aucun token de fournisseur dans le module ;
- aucune donnée client ou transactionnelle dans le payload ;
- aucun label de métrique par tenant, transaction, batch ou fichier ;
- aucune suppression du cooldown pour contourner la déduplication ;
- aucun sink externe activé par défaut.

## Résilience

Le dispatcher propage l’échec du sink.

Le composant qui déclenchera le cycle devra décider ultérieurement de :

- retry avec backoff ;
- circuit breaker ;
- dead-letter queue ;
- journal d’incident ;
- escalade opérationnelle.

Ces mécanismes ne doivent pas être introduits dans la politique SLO elle-même.

## Tests attendus

La tranche couvre au minimum :

1. `NO_DATA` sans livraison ;
2. livraison d’un `WARNING` ;
3. payload borné avec SLI et breach ;
4. déduplication pendant cooldown ;
5. livraison de `RECOVERED` ;
6. propagation d’une erreur du sink ;
7. cycle orchestré complet sans données ;
8. cycle orchestré produisant `CRITICAL` ;
9. seuils SLO personnalisés ;
10. cooldown personnalisé.

## Implémentation runtime

Nouveaux composants :

```text
ReconciliationAlertSink
RECONCILIATION_ALERT_SINK
NoopReconciliationAlertSink
ReconciliationAlertDispatcher
ReconciliationMonitoringOrchestrator
```

Ils sont enregistrés dans `ReconciliationModule`.

`ReconciliationMonitoringOrchestrator` et `ReconciliationAlertDispatcher` sont exportés afin qu’un futur worker ou module d’exploitation puisse les consommer.

## Hors périmètre

Cette tranche ne fournit pas encore :

- scheduler périodique ;
- état de cooldown distribué ;
- sink PagerDuty ;
- sink Slack ;
- sink e-mail/SMS ;
- retry réseau ;
- DLQ ;
- routage par équipe ou pays ;
- astreinte ;
- stockage historique des alertes ;
- burn-rate multi-fenêtres.

## Prochaine tranche recommandée

La suite logique est de rendre l’état de déduplication compatible multi-réplique via un port de stockage d’état borné, sans imposer Redis dans le domaine.

Le contrat devrait permettre :

```text
load(key)
compare-and-set / save
reset(key)
```

avec une clé fixe de domaine ou une cardinalité strictement bornée, afin d’éviter qu’un déploiement horizontal génère plusieurs notifications identiques pour le même état SLO.
