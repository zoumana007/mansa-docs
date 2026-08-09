# Moteur de rapprochement financier

## 1. Objet

Ce document définit le comportement commun du moteur de rapprochement Mansa. Il sert à comparer les transactions internes avec les observations provenant des banques, acquéreurs, réseaux carte, opérateurs Mobile Money, partenaires publics, prestataires de paiement ou autres systèmes externes.

Le rapprochement ne modifie jamais le grand livre pour « faire correspondre » artificiellement les données. Il détecte, classe et expose les écarts. Toute correction financière réelle passe ensuite par un flux métier explicite et, si nécessaire, une écriture compensatoire dans le ledger.

## 2. Sources comparées

Le moteur travaille au minimum avec deux vues :

- `internal` : transaction telle qu'enregistrée par Mansa ;
- `provider` : transaction telle qu'observée chez le fournisseur ou dans un fichier de compensation.

Chaque snapshot comparable contient au minimum :

```text
reference
amountMinor
currency
status
```

Les montants sont exprimés en unités mineures entières. Les devises sont normalisées en codes ISO à trois lettres. Les statuts fournisseur sont normalisés par l'adaptateur du fournisseur avant comparaison lorsque la taxonomie externe diffère de la taxonomie interne.

## 3. Résultat déterministe

Une même paire de snapshots doit toujours produire le même résultat. L'ordre de priorité des écarts est :

1. transaction interne absente ;
2. transaction fournisseur absente ;
3. référence fournisseur dupliquée ;
4. devise différente ;
5. montant différent ;
6. statut différent ;
7. correspondance complète.

Les motifs contractuels sont :

```text
MISSING_INTERNAL_TRANSACTION
MISSING_PROVIDER_TRANSACTION
DUPLICATE_PROVIDER_TRANSACTION
CURRENCY_MISMATCH
AMOUNT_MISMATCH
STATUS_MISMATCH
OTHER
```

Une transaction sans écart prend le statut `MATCHED`. Un écart automatique prend le statut `MISMATCHED` jusqu'à résolution, exception documentée ou nouvel import.

## 4. Doublons fournisseur

La détection de doublon est réalisée avant les comparaisons de devise, montant et statut. Une référence fournisseur apparaissant plusieurs fois dans une même source ne doit jamais être silencieusement agrégée.

Le pipeline d'import doit conserver :

- la référence brute ;
- le nombre d'occurrences ;
- la source ;
- le lot ;
- l'empreinte ou identifiant de ligne si disponible ;
- les horodatages d'import.

La résolution d'un doublon doit être auditée.

## 5. Transactions manquantes

`MISSING_INTERNAL_TRANSACTION` indique qu'une opération existe chez le fournisseur mais n'a pas de correspondance interne.

`MISSING_PROVIDER_TRANSACTION` indique qu'une opération interne attendue n'apparaît pas dans la source fournisseur pour la fenêtre de rapprochement considérée.

Une absence ne doit être déclarée qu'après application de la fenêtre temporelle, du fuseau horaire, des délais de compensation et des tolérances contractuelles propres au fournisseur.

## 6. Devise

La devise est comparée avant le montant. Une transaction XOF et une transaction EUR ne doivent jamais être déclarées identiques même si leurs valeurs numériques sont égales.

Le rapprochement multi-devise doit conserver séparément :

- devise d'origine ;
- devise de règlement ;
- taux de change appliqué ;
- frais ;
- référence de conversion ;
- date de valeur.

La première tranche exécutable compare une devise normalisée unique par snapshot. Les flux FX complexes seront rapprochés avec leurs propres jambes comptables.

## 7. Montant

Les comparaisons de montant utilisent exclusivement les unités mineures entières. Aucun `float` métier n'est accepté.

Une différence de frais ne doit pas être cachée dans une tolérance générique. Les frais fournisseur doivent être représentés explicitement dans le modèle de règlement ou dans des écritures dédiées.

## 8. Statut

Les statuts sont normalisés en majuscules avant comparaison. Les adaptateurs partenaires doivent convertir leurs états vers une taxonomie Mansa documentée avant de faire appel au moteur lorsque des synonymes existent.

Exemples d'écarts à traiter :

- Mansa `SETTLED`, fournisseur `FAILED` ;
- Mansa `REFUNDED`, fournisseur `SETTLED` ;
- Mansa `PENDING`, fournisseur `SETTLED` au-delà du délai normal de propagation.

## 9. Lots

Chaque exécution produit ou alimente un lot de rapprochement. Un lot doit pouvoir exposer :

- nombre total d'éléments ;
- éléments rapprochés ;
- éléments en écart ;
- répartition des écarts par motif ;
- éléments résolus ;
- éléments ignorés avec justification ;
- période couverte ;
- fournisseur ;
- source importée ;
- dates de début et de fin ;
- statut global.

Le contrat `ReconciliationBatchSummary` reste la vue API principale du lot.

## 10. Résolution manuelle

Seuls les éléments non résolus peuvent être marqués `RESOLVED` ou `IGNORED`.

Toute résolution exige :

- acteur authentifié ;
- permission spécifique ;
- justification non vide ;
- code de motif ;
- clé d'idempotence ;
- corrélation ;
- date ;
- journal d'audit.

`IGNORED` ne signifie pas que l'écart disparaît. L'écart brut et sa cause restent conservés pour audit et reporting.

## 11. Immutabilité et recalcul

Le résultat d'un rapprochement doit être reproductible à partir des données source conservées ou de leurs représentations auditées.

Une correction ne réécrit pas silencieusement un ancien résultat. Selon le cas, le système crée une nouvelle exécution, une résolution auditée ou un rapprochement supersédant l'ancien.

## 12. Sécurité

Les imports fournisseurs sont des entrées non fiables. Ils doivent être :

- limités en taille ;
- validés au niveau schéma ;
- protégés contre formules et contenus actifs dans les formats tabulaires ;
- analysés avant stockage si fichiers ;
- associés à un fournisseur et une organisation ;
- traités de façon idempotente ;
- isolés par tenant ;
- tracés avec corrélation.

Aucun identifiant bancaire sensible, secret API ou donnée carte complète ne doit apparaître dans les logs de rapprochement.

## 13. Observabilité

Les métriques minimales sont :

```text
reconciliation_batches_total
reconciliation_items_total
reconciliation_matched_total
reconciliation_mismatched_total{reason}
reconciliation_resolution_total{status}
reconciliation_batch_duration_seconds
reconciliation_import_failures_total{provider}
```

Des alertes doivent pouvoir être configurées sur le taux d'écart, l'absence de fichier attendu, la durée de traitement, les doublons et les erreurs répétées d'un fournisseur.

## 14. Implémentation de référence

Le contrat métier est maintenu dans :

`mansa-platform/packages/contracts/src/reconciliation.ts`

Le contrat de transport est maintenu dans :

`mansa-platform/packages/contracts/src/reconciliation-api.ts`

Le moteur de comparaison expose une fonction pure `compareReconciliationTransactions` ainsi qu'un résumé de lot `summarizeReconciliationComparisons`.

La persistance PostgreSQL de référence est désormais définie dans :

`mansa-platform/apps/api-gateway/prisma/schema.prisma`

avec les modèles :

```text
ReconciliationBatch
ReconciliationItem
```

et les enums persistés alignés sur les contrats :

```text
ReconciliationBatchStatus
ReconciliationItemStatus
ReconciliationMismatchReason
```

La migration correspondante est versionnée dans :

`apps/api-gateway/prisma/migrations/20260809073500_add_reconciliation_persistence/migration.sql`.

Le modèle de lot conserve le fournisseur, la référence et l'empreinte de source, la période couverte, le statut, les compteurs, les dates de traitement et la cause d'échec. L'unicité `(providerId, sourceFingerprint)` empêche de créer silencieusement deux lots pour exactement la même source fournisseur lorsque son empreinte est disponible.

Le modèle d'item conserve séparément références, montants, statuts interne/fournisseur, nombre d'occurrences fournisseur, motif d'écart, empreinte de ligne et données de résolution. La clé `resolutionIdempotencyKey` est unique lorsqu'elle est fournie afin qu'une même résolution ne soit pas appliquée plusieurs fois.

La tranche implémentée couvre maintenant :

- normalisation devise/statut ;
- validation des montants entiers ;
- transactions manquantes dans les deux sens ;
- doublons de référence fournisseur ;
- écart de devise ;
- écart de montant ;
- écart de statut ;
- résumé déterministe par motif ;
- schéma PostgreSQL des lots et items ;
- migration reproductible et index de consultation ;
- champs nécessaires à l'idempotence d'import et de résolution.

## 15. Contraintes de persistance

Les règles suivantes sont obligatoires :

1. `periodEnd` doit être postérieur ou égal à `periodStart` ; cette validation est assurée par le service métier avant écriture et doit être couverte par tests ;
2. tous les montants sont stockés en `BIGINT` et convertis de façon sûre aux frontières TypeScript ;
3. une ligne ne doit pas être créée sans référence interne ni référence fournisseur ;
4. `providerOccurrenceCount` doit rester supérieur ou égal à 1 ;
5. une résolution ne doit pas supprimer le motif d'écart historique ;
6. le recalcul des compteurs du lot doit être transactionnel avec les changements d'état des items lorsque ces compteurs sont matérialisés ;
7. les imports doivent rechercher un lot existant par fournisseur et empreinte avant création ;
8. les données brutes sensibles ne doivent pas être dupliquées dans `metadata` lorsque des références minimales suffisent.

Les contraintes impossibles à exprimer proprement dans Prisma sans rigidifier la migration restent validées dans le service applicatif puis dans les tests PostgreSQL.

## 16. Tests de recette minimaux

Le lot est acceptable lorsque les tests automatisés démontrent :

1. une paire identique devient `MATCHED` ;
2. une transaction absente de chaque côté est classée correctement ;
3. un doublon fournisseur est prioritaire sur les autres écarts ;
4. une devise différente est détectée avant le montant ;
5. un montant différent est détecté ;
6. un statut différent est détecté ;
7. les snapshots invalides sont rejetés ;
8. le résumé du lot compte exactement les résultats et motifs ;
9. les entrées originales ne sont pas mutées par le moteur ;
10. la migration crée les deux tables et leurs index ;
11. une même empreinte de source ne peut pas créer deux lots pour le même fournisseur ;
12. deux fournisseurs différents peuvent utiliser la même empreinte sans collision ;
13. une même clé de résolution ne peut être appliquée à deux items ;
14. la suppression d'un lot possédant des items est refusée par la relation `RESTRICT`.

## 17. Étapes suivantes

La tranche suivante doit ajouter les repositories Prisma des lots et items, une transaction de création/import de lot, puis un import fournisseur de test utilisant une source tabulaire fictive. Elle devra recalculer les compteurs de lot de façon atomique et couvrir l'idempotence d'import.

Avant production, il restera à ajouter les tests PostgreSQL/concurrence, les métriques exportées, les seuils d'alerte, les adaptateurs partenaires réels et les runbooks de reprise.
