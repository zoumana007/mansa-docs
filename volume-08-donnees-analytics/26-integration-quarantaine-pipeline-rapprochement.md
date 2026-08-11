# 26 — Intégration de la quarantaine dans le pipeline de rapprochement

## Objectif

Cette tranche branche effectivement `ReconciliationIngestionBoundary` dans `ReconciliationImportService` avant toute résolution d’adaptateur fournisseur et avant toute persistance.

Le flux réel devient :

```text
source reçue
→ monitoring import started
→ validation provider-neutral
→ quarantaine bornée si invalide
→ résolution de l’adaptateur
→ préparation normalisée
→ repository
→ résumé métier
→ monitoring succès/échec
```

## Ordre de sécurité

La validation doit précéder :

- `providerRegistry.resolve(...)` ;
- toute logique spécifique à un fournisseur ;
- toute écriture en base ;
- toute future émission vers un connecteur externe.

Une source invalide doit donc échouer sans invoquer le repository et sans dépendre de l’existence d’un adaptateur fournisseur.

## Erreur de quarantaine

Le service expose désormais une erreur structurée `ReconciliationIngestionQuarantineError` contenant uniquement :

- un code fermé de rejet ;
- éventuellement l’index de ligne fautive ;
- l’empreinte technique bornée de la source ;
- le nombre de lignes.

Elle ne contient pas le contenu brut de la ligne, du fichier ou des secrets fournisseur.

## Observabilité

Une quarantaine est comptabilisée comme import en échec dans le monitoring actuel :

```text
recordImportStarted()
→ validation
→ rejet
→ recordImportFailed(...)
```

Cela garantit qu’un rejet à la frontière n’échappe pas aux indicateurs opérationnels existants.

La prochaine évolution recommandée est d’ajouter un compteur dédié de quarantaine à faible cardinalité, indexé uniquement par code fermé de rejet, sans `providerId`, `organizationId`, référence de fichier ou autre dimension non bornée.

## Tests

Les tests couvrent maintenant deux niveaux :

1. tests unitaires de la frontière de validation ;
2. test d’intégration du service d’import vérifiant qu’une source malformée est rejetée avant résolution d’adaptateur ou persistance.

Le test d’intégration vérifie notamment :

- aucune écriture repository ;
- erreur typée de quarantaine ;
- code `INVALID_AMOUNT` ;
- index de ligne ;
- empreinte SHA-256 bornée ;
- comptabilisation `started` puis `failed`.

## Compatibilité

L’entrée historique `importTestProviderSource(...)` reste une façade transitoire qui délègue vers le pipeline provider-neutral. Elle bénéficie donc automatiquement de la même frontière de validation.

## Hors périmètre de cette tranche

Cette tranche n’introduit volontairement pas :

- stockage durable de fichiers rejetés ;
- DLQ ;
- S3/Blob Storage ;
- chiffrement de payload de quarantaine ;
- interface opérateur ;
- politique de rétention ;
- reprise manuelle ;
- compteur dédié par code.

Ces sujets nécessitent une politique de données et d’exploitation explicite avant implémentation.
