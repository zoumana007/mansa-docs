# 31 — Garde de persistance de quarantaine injectable

## Objet

Cette tranche durcit la séparation entre la décision par fournisseur et la garde technique globale qui interdit actuellement la persistance brute des sources rejetées pendant le rapprochement.

Jusqu’ici, `ReconciliationQuarantineDecisionService` instancait directement `ReconciliationQuarantinePersistencePolicy`. Cette approche conservait bien le comportement fail-closed, mais créait une dépendance locale difficile à partager, remplacer ou auditer au niveau du conteneur NestJS.

## État implémenté

Dans `mansa-platform` :

- `ReconciliationQuarantinePersistencePolicy` est désormais un provider NestJS injectable ;
- `ReconciliationModule` l’enregistre explicitement ;
- `ReconciliationQuarantineDecisionService` reçoit la garde par injection de dépendances ;
- le provider est exporté par le module pour les futurs consommateurs internes ;
- l’instanciation directe reste possible dans les tests unitaires, avec `SIGNALS_ONLY` comme mode fermé par défaut.

## Invariant de sécurité

Aucun changement n’ouvre le stockage brut.

Le plan effectif reste :

```text
mode = SIGNALS_ONLY
persistRawSource = false
persistProviderPayload = false
manualReplayAllowed = false
durableMetadataAllowed = false
```

Toute tentative de construire la garde en mode `RAW_SOURCE` continue à lever une erreur.

## Pourquoi l’injection est importante

La garde de persistance est une politique transverse. Elle ne doit pas être recréée silencieusement par plusieurs services si, plus tard, elle doit porter une configuration contrôlée, une preuve de déploiement, un état de feature flag de sécurité ou une dépendance KMS/HSM.

Le conteneur NestJS devient le point d’assemblage explicite :

```text
ReconciliationModule
  → ReconciliationQuarantinePersistencePolicy
  → ReconciliationQuarantineDecisionService
```

Cela permet aux futurs composants de partager la même politique technique sans contourner la frontière de sécurité.

## Compatibilité tests

Le constructeur de `ReconciliationQuarantineDecisionService` conserve une valeur par défaut fermée pour les tests qui instancient directement le service sans conteneur NestJS.

Cette valeur par défaut ne doit jamais activer `RAW_SOURCE`.

## Règles pour une future ouverture RAW_SOURCE

Le simple fait de rendre la politique injectable ne constitue aucune autorisation de stockage brut.

Une évolution vers `RAW_SOURCE` nécessitera au minimum :

- approbation explicite de gouvernance ;
- rétention bornée par fournisseur ;
- chiffrement en transit et au repos ;
- clés gérées par KMS/HSM ;
- contrôle d’accès par rôle et séparation des responsabilités ;
- journal d’accès immuable ;
- suppression vérifiable ;
- procédure de replay manuel contrôlée ;
- métriques et alertes ;
- tests négatifs démontrant qu’un fournisseur non approuvé reste en `SIGNALS_ONLY` ;
- procédure de rollback immédiat vers `SIGNALS_ONLY`.

## Cohérence avec la tranche précédente

Cette tranche prolonge `30-decision-quarantaine-fournisseur-rapprochement.md` sans modifier ses décisions :

```text
politique fournisseur
→ décision fournisseur
→ garde technique globale injectable
→ mode effectif
```

La garde technique conserve le dernier mot.

## Prochaine tranche recommandée

Ajouter un test d’intégration NestJS du module de rapprochement vérifiant que `ReconciliationQuarantineDecisionService` et `ReconciliationQuarantinePersistencePolicy` sont résolus par le conteneur et que la décision obtenue reste fail-closed pour un fournisseur inconnu et pour une politique `RAW_SOURCE` approuvée mais non activée techniquement.
