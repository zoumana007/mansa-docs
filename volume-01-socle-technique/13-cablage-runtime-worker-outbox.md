# Câblage runtime du worker outbox

## 1. Objet

Ce document précise comment le worker de livraison de l’outbox ledger est intégré au cycle de vie du backend Mansa sans imposer prématurément un broker particulier.

Le principe de sécurité opérationnelle est le suivant : le worker est désactivé par défaut et ne peut pas être activé tant qu’un publisher réel n’a pas été explicitement configuré dans l’infrastructure.

## 2. Composants de référence

Le dépôt `mansa-platform` contient :

- `apps/api-gateway/src/ledger-outbox-worker.ts` : cadence périodique et protection contre les chevauchements locaux ;
- `apps/api-gateway/src/ledger-outbox-dispatcher.service.ts` : prise en charge d’un lot, publication, succès, échec et backoff ;
- `apps/api-gateway/src/ledger-outbox-publisher.provider.ts` : token d’injection et binding du publisher ;
- `apps/api-gateway/src/ledger-outbox-lifecycle.service.ts` : démarrage et arrêt du worker avec le cycle de vie NestJS ;
- `apps/api-gateway/src/runtime-config.ts` : validation stricte de la configuration runtime ;
- `apps/api-gateway/src/ledger.module.ts` : enregistrement du lifecycle et du binding par défaut.

## 3. Désactivation par défaut

`LEDGER_OUTBOX_WORKER_ENABLED` vaut `false` par défaut.

Tant que cette valeur reste à `false`, le backend peut démarrer sans broker. Aucun timer de livraison outbox n’est créé et aucun événement n’est marqué comme publié artificiellement.

Cette règle permet de développer et tester le ledger sans introduire de transport fictif pouvant compromettre la sémantique financière.

## 4. Activation contrôlée

Pour activer le worker, l’infrastructure doit :

1. fournir une implémentation réelle de `LedgerOutboxPublisher` ;
2. remplacer le binding `UNCONFIGURED_LEDGER_OUTBOX_PUBLISHER` par un binding `configured: true` ;
3. fournir les secrets du broker via le gestionnaire de secrets de l’environnement ;
4. définir `LEDGER_OUTBOX_WORKER_ENABLED=true` ;
5. valider les timeouts, TLS, authentification workload et règles de retry ;
6. vérifier les métriques et alertes avant passage en production.

Si le worker est activé alors que le binding reste non configuré, le processus doit refuser de démarrer. Il est interdit de basculer silencieusement vers un publisher no-op qui marquerait les événements comme livrés.

## 5. Variables runtime

Les variables supportées sont :

```text
LEDGER_OUTBOX_WORKER_ENABLED=false
LEDGER_OUTBOX_INTERVAL_MS=1000
LEDGER_OUTBOX_BATCH_SIZE=50
LEDGER_OUTBOX_LEASE_MS=30000
LEDGER_OUTBOX_MAX_ATTEMPTS=10
LEDGER_OUTBOX_BASE_RETRY_DELAY_MS=1000
LEDGER_OUTBOX_MAX_RETRY_DELAY_MS=60000
LEDGER_OUTBOX_JITTER_RATIO=0.2
```

Toutes les valeurs numériques positives sont validées au démarrage. Le ratio de jitter doit rester compris entre `0` et `1`.

## 6. Cycle de vie

Au `OnModuleInit`, `LedgerOutboxLifecycleService` :

- charge la configuration runtime ;
- quitte immédiatement si le worker est désactivé ;
- vérifie qu’un publisher réel est déclaré ;
- construit `LedgerOutboxWorker` avec les limites validées ;
- démarre le timer.

Au `OnApplicationShutdown`, le timer est arrêté et la référence locale est libérée.

Le lifecycle expose également un état local `isStarted()` destiné aux tests et à la future intégration de santé interne.

## 7. Mapping de configuration

La variable `LEDGER_OUTBOX_BATCH_SIZE` alimente l’option `limit` de `claimBatch`.

Les autres paramètres sont transmis sans changer leur sens :

- `leaseMs` : durée du bail de réclamation ;
- `maxAttempts` : budget de tentatives avant dead-letter opérationnelle ;
- `baseRetryDelayMs` : délai de base ;
- `maxRetryDelayMs` : plafond du backoff ;
- `jitterRatio` : dispersion aléatoire bornée ;
- `intervalMs` : cadence du worker.

## 8. Contrat du publisher

Le publisher concret doit respecter `LedgerOutboxPublisher` :

```ts
interface LedgerOutboxPublisher {
  publish(event: ClaimedOutboxEvent): Promise<void>;
}
```

Une promesse résolue signifie que le transport a confirmé la prise en charge selon le contrat de l’adaptateur. Une erreur ou un timeout doit rejeter la promesse afin que le dispatcher conserve l’événement en retry.

Le publisher ne doit jamais :

- appeler directement `markPublished` ;
- supprimer l’événement SQL ;
- modifier le payload ;
- masquer un échec du broker ;
- embarquer un secret dans le code ou les logs.

## 9. Tests obligatoires

Le dépôt plateforme doit couvrir au minimum :

- worker désactivé : aucun démarrage ;
- worker activé sans publisher : échec explicite ;
- publisher configuré : démarrage puis arrêt propres ;
- valeurs runtime par défaut ;
- valeurs runtime personnalisées ;
- rejet des entiers invalides ;
- rejet d’un jitter hors bornes.

Ces tests complètent les tests existants du worker, du dispatcher, du service outbox et des routes d’exploitation.

## 10. Prochaine étape

Le prochain lot infrastructure doit fournir un premier adaptateur de broker réel derrière le binding, sans modifier les services métier. Le choix final peut rester Kafka, RabbitMQ, NATS ou un service cloud équivalent tant que les exigences de durabilité, sécurité, idempotence, observabilité et exploitation sont respectées.

Avant production restent également nécessaires : export métriques/alertes, tests PostgreSQL de concurrence, tests de reprise après crash, audit des remises en file dead-letter et runbook d’incident.
