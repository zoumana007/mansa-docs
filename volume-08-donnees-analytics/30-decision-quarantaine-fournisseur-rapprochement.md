# 30 — Décision fail-closed de quarantaine fournisseur pour le rapprochement

## Objet

Cette tranche prolonge le registre de politiques par fournisseur défini dans `29-registre-politiques-quarantaine-fournisseurs-rapprochement.md`.

Elle introduit une couche de décision explicite entre le registre de politiques et la politique globale de persistance afin qu’une approbation métier ou sécurité d’un fournisseur ne soit jamais interprétée, à elle seule, comme une autorisation technique de stocker une source brute rejetée.

## État implémenté

Dans `mansa-platform`, `ReconciliationQuarantineDecisionService` est désormais disponible dans `ReconciliationModule`.

Le service reçoit un `providerId`, consulte `ReconciliationQuarantinePolicyRegistry` et produit une décision bornée contenant uniquement :

- le fournisseur normalisé ;
- la résolution : `APPROVED` ou `FALLBACK` ;
- le mode demandé : `SIGNALS_ONLY` ou `RAW_SOURCE` ;
- le mode effectif ;
- l’autorisation ou non de stockage brut ;
- l’autorisation ou non de replay manuel ;
- une raison de décision à faible cardinalité.

Aucun payload fournisseur, aucune ligne rejetée, aucun secret et aucune donnée métier sensible ne sont ajoutés à cette décision.

## Invariant principal

Le mode effectif reste actuellement :

```text
SIGNALS_ONLY
```

même si une politique fournisseur `RAW_SOURCE` est enregistrée et approuvée.

La raison est volontaire : `ReconciliationQuarantinePersistencePolicy` continue à bloquer le stockage brut tant que les garanties techniques restantes ne sont pas implémentées et vérifiées.

Ainsi :

```text
politique fournisseur RAW_SOURCE approuvée
→ décision fournisseur = APPROVED
→ mode demandé = RAW_SOURCE
→ politique globale de persistance = SIGNALS_ONLY
→ mode effectif = SIGNALS_ONLY
→ aucun stockage brut
```

## Fournisseur inconnu

Un fournisseur sans politique enregistrée est traité sans ouverture implicite :

```text
providerId inconnu
→ résolution FALLBACK
→ requestedMode SIGNALS_ONLY
→ effectiveMode SIGNALS_ONLY
→ rawSourcePersistenceAllowed = false
→ manualReplayAllowed = false
```

Ce comportement évite qu’une absence de configuration devienne une permission.

## Politique non approuvée

Une politique enregistrée en état `DRAFT` ou `SUSPENDED` ne devient pas active par simple présence dans le registre.

La couche de décision la ramène vers le fallback fermé `SIGNALS_ONLY`.

Le consommateur n’a donc pas besoin de déduire lui-même si un brouillon peut être utilisé.

## Politique SIGNALS_ONLY approuvée

Une politique approuvée en `SIGNALS_ONLY` produit :

- `resolution = APPROVED` ;
- `requestedMode = SIGNALS_ONLY` ;
- `effectiveMode = SIGNALS_ONLY` ;
- `rawSourcePersistenceAllowed = false` ;
- `manualReplayAllowed = false` ;
- raison `APPROVED_SIGNALS_ONLY`.

## Politique RAW_SOURCE approuvée

Une politique `RAW_SOURCE` cohérente et approuvée est reconnue mais reste techniquement fermée :

- `resolution = APPROVED` ;
- `requestedMode = RAW_SOURCE` ;
- `effectiveMode = SIGNALS_ONLY` ;
- `rawSourcePersistenceAllowed = false` ;
- `manualReplayAllowed = false` ;
- raison `APPROVED_RAW_SOURCE_NOT_TECHNICALLY_ENABLED`.

Cette distinction est importante pour l’exploitation : elle permet de savoir qu’une décision de gouvernance existe sans prétendre que l’infrastructure de stockage sécurisé est prête.

## Raisons de décision

Les raisons restent volontairement bornées :

```text
APPROVED_SIGNALS_ONLY
APPROVED_RAW_SOURCE_NOT_TECHNICALLY_ENABLED
NO_APPROVED_PROVIDER_POLICY
```

Elles peuvent être utilisées plus tard pour des métriques à faible cardinalité et des alertes sans introduire le nom d’un fichier, une référence transactionnelle ou un contenu fournisseur dans les labels.

## Séparation des responsabilités

La chaîne attendue est désormais :

```text
ReconciliationQuarantinePolicyRegistry
→ décrit la décision approuvée par fournisseur

ReconciliationQuarantineDecisionService
→ combine cette décision avec l’état technique global

ReconciliationQuarantinePersistencePolicy
→ reste la garde technique finale de stockage
```

Aucune de ces couches ne doit être court-circuitée.

## Tests ajoutés

Les tests couvrent :

- fallback `SIGNALS_ONLY` pour fournisseur inconnu ;
- décision approuvée `SIGNALS_ONLY` ;
- maintien du blocage brut malgré une politique `RAW_SOURCE` approuvée ;
- fallback pour une politique enregistrée mais non approuvée ;
- immutabilité de la décision renvoyée.

## Ce que cette tranche n’active pas

Cette tranche n’ajoute pas :

- de stockage de payload brut ;
- de bucket objet ;
- de table de quarantaine brute ;
- de chiffrement applicatif ;
- de KMS/HSM réel ;
- de replay automatique ;
- de téléchargement de fichier rejeté ;
- d’endpoint administratif d’accès aux quarantaines brutes.

Elle prépare la frontière de décision sans augmenter la surface de données conservées.

## Prochaine tranche recommandée

Brancher `ReconciliationQuarantineDecisionService` sur le chemin de rejet de `ReconciliationImportService` afin d’attacher une décision bornée à l’erreur de quarantaine et à l’observabilité, sans modifier le comportement de persistance actuel. Les tests devront vérifier que les sources malformées restent rejetées avant résolution de l’adaptateur et qu’aucun payload brut n’est conservé.
