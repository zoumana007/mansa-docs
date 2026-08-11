# 27 — Métriques de quarantaine d’ingestion du rapprochement

## Objet

Cette tranche complète la frontière de validation et la quarantaine du pipeline de rapprochement avec une observabilité dédiée à faible cardinalité.

Elle prolonge :

- `25-frontiere-validation-quarantaine-ingestion-rapprochement.md` ;
- `26-integration-quarantaine-pipeline-rapprochement.md`.

Une source rejetée reste comptabilisée comme import en échec, mais elle est désormais aussi comptabilisée explicitement comme quarantaine afin de distinguer les problèmes de qualité d’entrée des autres échecs techniques ou fournisseur.

## État implémenté

Dans `mansa-platform` :

- `ReconciliationImportService` appelle `recordImportQuarantined(code)` immédiatement après la décision de rejet provider-neutral et avant de lever `ReconciliationIngestionQuarantineError` ;
- le `catch` existant continue ensuite d’enregistrer l’import comme échoué ;
- `ReconciliationOperationalMonitor` maintient un compteur total de quarantaines et un compteur par code fermé de rejet ;
- `LowCardinalityReconciliationMetricsExporter` exporte ces compteurs sous des noms de métriques fixes, sans labels dynamiques.

Une quarantaine est donc un sous-ensemble explicite des imports en échec et non une troisième issue indépendante du pipeline.

## Invariant de comptage

Pour une source rejetée à la frontière :

```text
imports_started += 1
imports_quarantined += 1
quarantine_<reason> += 1
imports_failed += 1
```

Elle ne doit jamais incrémenter :

- `imports_succeeded` ;
- `imported_items` ;
- `matched_items` ;
- `mismatched_items`.

## Codes bornés

Les seuls motifs exportables sont les codes fermés du domaine d’ingestion :

- `PROVIDER_ID_REQUIRED` ;
- `INVALID_PERIOD` ;
- `EMPTY_SOURCE` ;
- `SOURCE_TOO_LARGE` ;
- `INVALID_PROVIDER_REFERENCE` ;
- `INVALID_AMOUNT` ;
- `INVALID_CURRENCY` ;
- `INVALID_STATUS`.

Aucune chaîne arbitraire provenant d’un fichier fournisseur ne peut devenir une dimension de métrique.

## Métriques

Compteur total :

- `mansa_reconciliation_imports_quarantined_total`

Compteurs fixes par motif :

- `mansa_reconciliation_quarantine_provider_id_required_total`
- `mansa_reconciliation_quarantine_invalid_period_total`
- `mansa_reconciliation_quarantine_empty_source_total`
- `mansa_reconciliation_quarantine_source_too_large_total`
- `mansa_reconciliation_quarantine_invalid_provider_reference_total`
- `mansa_reconciliation_quarantine_invalid_amount_total`
- `mansa_reconciliation_quarantine_invalid_currency_total`
- `mansa_reconciliation_quarantine_invalid_status_total`

## Confidentialité

Ces métriques ne contiennent aucun :

- `organizationId` ;
- `providerId` ;
- nom de fichier ;
- référence fournisseur ;
- index de ligne ;
- fingerprint ;
- payload brut ;
- identifiant de transaction.

Le code de rejet est représenté par le nom fixe de la série et non par un label.

## Tests

Les tests couvrent :

- cumul du compteur total de quarantaine ;
- cumul par code fermé ;
- absence d’incrément sur les autres codes ;
- propagation de `INVALID_AMOUNT` depuis le pipeline réel d’import ;
- ordre `started → quarantined → failed` ;
- présence des huit métriques fixes de motifs ;
- absence de labels et d’identifiants sensibles dans les samples exportés.

## Limites

Cette tranche reste process-local. Elle ne remplace pas :

- la persistance d’événements d’exploitation ;
- un backend de métriques externe ;
- les runbooks d’incident ;
- une politique de stockage des fichiers rejetés ;
- une interface de reprise manuelle.

Aucun payload rejeté n’est conservé par ce mécanisme.

## Prochaine tranche recommandée

Définir la politique opérationnelle de quarantaine durable avant tout stockage de source rejetée : classification des données, chiffrement, rétention, suppression, droits d’accès, audit et procédure de reprise. Tant que cette politique n’est pas validée, la plateforme doit conserver uniquement les signaux bornés déjà disponibles et ne pas introduire de bucket ou DLQ contenant des données fournisseur brutes.
