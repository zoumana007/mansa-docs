# 32 — Test de la garde injectée de quarantaine

## Objet

Cette tranche verrouille par test le comportement fail-closed lorsque la garde globale de persistance est injectée explicitement dans `ReconciliationQuarantineDecisionService`.

La tranche précédente a rendu `ReconciliationQuarantinePersistencePolicy` injectable dans le module NestJS. Il fallait maintenant démontrer qu’une politique fournisseur déclarée `RAW_SOURCE` et approuvée dans le registre ne suffit toujours pas à autoriser le stockage brut.

## État implémenté

Dans `mansa-platform`, le test `apps/api-gateway/test/reconciliation-quarantine-persistence-policy.test.mjs` couvre désormais quatre invariants :

- le plan par défaut reste `SIGNALS_ONLY` et immuable ;
- la construction de la garde en mode `RAW_SOURCE` est refusée ;
- l’appel direct à la garde de persistance brute lève une erreur ;
- une garde injectée explicitement dans `ReconciliationQuarantineDecisionService` conserve une politique fournisseur `RAW_SOURCE` approuvée en mode effectif `SIGNALS_ONLY`.

## Invariant vérifié

Pour un fournisseur dont la politique est :

```text
status = APPROVED
mode = RAW_SOURCE
retentionDays = 7
replayStatus = MANUAL_REVIEW
```

la décision effective reste :

```text
resolution = APPROVED
requestedMode = RAW_SOURCE
effectiveMode = SIGNALS_ONLY
rawSourcePersistenceAllowed = false
manualReplayAllowed = false
reason = APPROVED_RAW_SOURCE_NOT_TECHNICALLY_ENABLED
```

Le registre fournisseur peut donc exprimer une intention gouvernée sans court-circuiter la garde technique globale.

## Pourquoi ce test est important

Deux couches restent volontairement séparées :

1. la politique métier et de gouvernance par fournisseur ;
2. l’autorisation technique globale de persister des données brutes.

Le passage de la première couche à `APPROVED` ne doit jamais activer implicitement la seconde.

Ce test protège contre une régression future où un développeur pourrait interpréter `RAW_SOURCE + APPROVED` comme une autorisation suffisante de stockage.

## Sécurité

Aucune donnée brute n’est ajoutée, aucun payload fournisseur n’est persisté et aucun replay n’est ouvert par cette tranche.

La garde reste fermée tant qu’une évolution dédiée n’a pas défini et validé :

- rétention bornée ;
- chiffrement ;
- KMS/HSM ;
- RBAC ;
- journal d’accès ;
- suppression vérifiable ;
- replay contrôlé ;
- preuves d’exploitation ;
- rollback vers `SIGNALS_ONLY`.

## Cohérence documentation / code

Cette tranche complète `31-garde-persistance-quarantaine-injectable.md` en ajoutant la preuve exécutable du comportement attendu lors de l’injection explicite de la garde.

## Prochaine tranche recommandée

Ajouter un contrôle d’assemblage du `ReconciliationModule` qui vérifie que la garde et le service de décision figurent bien dans les providers/export du module, sans nécessiter de connexion PostgreSQL lors du test. Cela permettra de protéger le câblage NestJS tout en évitant de transformer un test de sécurité de configuration en test d’infrastructure.