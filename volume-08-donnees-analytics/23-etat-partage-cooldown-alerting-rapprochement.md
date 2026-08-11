# 23 — État partagé et cooldown atomique de l’alerting rapprochement

## Objectif

Cette tranche retire la dépendance conceptuelle à un état de cooldown strictement local au processus pour l’alerting de rapprochement.

Le moteur de décision doit pouvoir être exécuté par plusieurs réplicas sans générer plusieurs alertes identiques pour la même transition ou le même rappel de cooldown.

La logique métier reste provider-neutral et ne dépend directement ni de Redis, ni de PostgreSQL, ni d’un fournisseur d’incident.

## Problème traité

Avant cette tranche, `ReconciliationAlertingPolicy` conservait :

- le dernier statut SLO ;
- la date de dernière notification ;

uniquement en mémoire dans l’instance du processus.

Avec plusieurs réplicas, deux workers pouvaient évaluer le même état `WARNING` au même instant et chacun décider qu’une notification devait être envoyée.

Le cooldown n’était donc pas globalement cohérent.

## Port d’état

Le domaine expose désormais un port :

```text
ReconciliationAlertStateStore
```

avec les opérations :

```text
transact(key, operation)
reset(key)
```

`transact` doit être atomique pour une même clé.

Une implémentation de production peut utiliser :

- Redis avec transaction/commande atomique ;
- PostgreSQL avec verrouillage de ligne ou mécanisme transactionnel approprié ;
- un autre stockage partagé offrant les mêmes garanties.

Le choix d’infrastructure ne doit pas modifier la politique de décision.

## État stocké

L’état minimal contient :

```text
status
lastNotificationAtMs
```

Aucun payload d’alerte, secret, contenu financier ou donnée personnelle ne doit être stocké dans ce port.

## Clé d’état

La clé par défaut actuelle est :

```text
reconciliation:alerting:global
```

Elle représente l’évaluation SLO agrégée actuelle du rapprochement.

Si le monitoring est ultérieurement segmenté par pays, fournisseur, rail ou organisation, la clé devra être dérivée explicitement du scope de métriques correspondant. Il est interdit de réutiliser une clé globale pour plusieurs agrégats indépendants.

## Politique de décision

`ReconciliationAlertingPolicy` conserve l’API locale synchrone historique pour les tests unitaires et les appels directs existants.

Le chemin de dispatch opérationnel utilise désormais une évaluation partagée atomique :

```text
evaluateShared(...)
```

Le comportement fonctionnel reste :

- `NO_DATA` : aucune notification ;
- transition vers `WARNING` : notification immédiate ;
- transition vers `CRITICAL` : notification immédiate ;
- passage d’un état dégradé à `HEALTHY` : `RECOVERED` ;
- état dégradé stable pendant le cooldown : aucune notification ;
- état dégradé stable après expiration du cooldown : `REMINDER`.

## Atomicité

La lecture de l’état, la décision et l’écriture du nouvel état doivent appartenir à une seule transaction logique.

Deux réplicas concurrents sur la même clé ne doivent pas pouvoir :

1. lire tous les deux le même ancien état ;
2. décider tous les deux qu’une alerte est éligible ;
3. écrire chacun une nouvelle date de notification ;
4. envoyer deux alertes équivalentes.

Le store doit sérialiser ou rendre atomique cette transition.

## Implémentation locale

`InMemoryReconciliationAlertStateStore` fournit une implémentation de développement et de test.

Elle sérialise les transactions concurrentes par clé dans un même processus.

Elle ne constitue pas un backend distribué de production. Son intérêt est :

- valider le contrat ;
- tester les courses concurrentes ;
- permettre au module de fonctionner sans infrastructure supplémentaire ;
- conserver la séparation entre domaine et technologie de stockage.

## Injection NestJS

`ReconciliationModule` expose le token :

```text
RECONCILIATION_ALERT_STATE_STORE
```

et le lie actuellement à l’implémentation mémoire.

Un déploiement multi-réplique réel devra remplacer ce binding par un adaptateur partagé sans modifier `ReconciliationAlertingPolicy` ni `ReconciliationAlertDispatcher`.

## Dispatcher

`ReconciliationAlertDispatcher` attend désormais la décision issue de `evaluateShared` avant d’appeler le sink.

Le sink reste indépendant du store d’état :

```text
politique de décision
→ état atomique
→ décision
→ dispatcher
→ sink provider-neutral
```

Cette séparation évite de confondre déduplication/cooldown et livraison de notification.

## Sécurité

Le store ne doit jamais contenir :

- credentials ;
- tokens d’API ;
- payloads partenaires ;
- identifiants bancaires ;
- numéros de carte ;
- données KYC ;
- détails transactionnels bruts.

Les clés de stockage doivent rester de faible cardinalité et dérivées de scopes autorisés.

Une erreur de stockage ne doit pas conduire à contourner silencieusement le cooldown en production. L’adaptateur partagé devra échouer explicitement selon une politique opérationnelle documentée.

## Tests attendus

La tranche couvre au minimum :

1. deux instances de politique partageant le même store ;
2. deux évaluations concurrentes `WARNING` ;
3. une seule décision `shouldNotify=true` ;
4. l’autre décision bloquée par `COOLDOWN_ACTIVE` ;
5. propagation d’une transition `WARNING -> HEALTHY` entre deux instances ;
6. reset partagé ;
7. conservation des tests historiques de la politique locale.

## Hors périmètre

Cette tranche n’ajoute pas encore :

- Redis ;
- table PostgreSQL dédiée ;
- TTL distribué ;
- fencing token ;
- verrou distribué spécifique ;
- haute disponibilité du backend d’état ;
- migration automatique entre backends ;
- segmentation du monitoring par fournisseur/tenant.

Ces choix devront être pris à partir de l’architecture de déploiement réelle et des contraintes d’exploitation.

## Évolution recommandée

La prochaine étape doit être l’un des deux axes suivants selon les informations disponibles :

1. implémenter un adaptateur partagé réel pour `ReconciliationAlertStateStore` si l’infrastructure cible est arrêtée ;
2. poursuivre le pipeline provider-neutral d’import du rapprochement et brancher le registre d’adaptateurs dans le service d’import, sans inventer de fournisseur réel.

Aucune intégration fournisseur réelle ne doit être codée sans format officiel, sandbox ou documentation contractuelle vérifiable.
