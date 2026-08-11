# 33 — Contrôle d’assemblage du module de quarantaine

## Objet

Cette tranche protège le câblage NestJS de la chaîne de décision de quarantaine sans ouvrir de connexion PostgreSQL et sans transformer un test de configuration en test d’infrastructure.

Elle fait suite à `32-test-garde-injectee-quarantaine.md`, qui vérifie le comportement fail-closed lorsqu’une garde de persistance est injectée explicitement.

## État implémenté

Dans `mansa-platform`, le test :

`apps/api-gateway/test/reconciliation-module-wiring.test.mjs`

vérifie désormais que `ReconciliationModule` conserve dans ses `providers` :

- `ReconciliationQuarantinePolicyRegistry` ;
- `ReconciliationQuarantinePersistencePolicy` ;
- `ReconciliationQuarantineDecisionService`.

Le même test vérifie que ces trois éléments restent également exposés dans les `exports` du module.

## Invariant d’injection

Le contrôle vérifie aussi que `ReconciliationQuarantineDecisionService` dépend bien d’une instance injectée de `ReconciliationQuarantinePersistencePolicy`.

Il interdit la régression suivante :

```ts
private readonly persistencePolicy = new ReconciliationQuarantinePersistencePolicy();
```

La garde doit rester injectée afin que le module fournisse un point de décision technique unique et contrôlable.

## Pourquoi un contrôle statique ciblé

Le but de cette tranche n’est pas de démarrer l’application ni Prisma.

Le test lit les sources du module et du service afin de protéger les invariants de câblage avec un coût minimal :

- aucune base de données ;
- aucun réseau ;
- aucun secret ;
- aucun fournisseur externe ;
- aucun effet de bord d’initialisation NestJS.

Cela complète les tests comportementaux déjà présents sans créer de dépendance à l’infrastructure.

## Sécurité

Cette tranche ne modifie aucune règle de persistance.

Le comportement reste :

```text
RAW_SOURCE demandé
+ politique fournisseur approuvée
+ garde globale non ouverte
=> SIGNALS_ONLY effectif
```

Aucun payload fournisseur rejeté n’est rendu persistant et aucun replay manuel n’est activé.

## Cohérence documentation / code

Le lot ajoute une preuve exécutable du câblage décrit dans les tranches 31 et 32 :

- registre fournisseur ;
- garde globale de persistance ;
- service de décision ;
- assemblage NestJS explicite.

## Prochaine tranche recommandée

Ajouter un inventaire opérationnel en lecture seule des politiques de quarantaine enregistrées, retourné sous forme de snapshot immuable et déterministe, afin de préparer l’audit et la supervision sans exposer de payload fournisseur ni modifier les décisions de persistance.
