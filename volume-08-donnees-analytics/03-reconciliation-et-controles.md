# Réconciliation financière et contrôles

## 1. Objectif

La réconciliation compare les opérations enregistrées par Mansa avec les relevés fournis par les banques, opérateurs Mobile Money, réseaux de cartes, processeurs TPE et autres partenaires. Elle détecte les opérations absentes, dupliquées ou incohérentes avant règlement définitif et clôture comptable.

## 2. Principes obligatoires

- Chaque import partenaire crée un lot identifiable par `batchId`.
- Le fichier ou objet source reste référencé sans exposer d’URL signée durable ni de secret.
- Une ligne de rapprochement possède au moins une référence interne ou partenaire.
- Les montants sont exprimés en unités mineures entières et ne sont jamais comparés en nombre flottant.
- Une anomalie n’est jamais supprimée : elle est résolue ou ignorée avec justification.
- Toute résolution manuelle exige une permission dédiée, une authentification forte, un motif et un journal d’audit.
- Une correction financière utilise un journal compensatoire du grand livre ; la réconciliation ne modifie jamais directement une écriture publiée.
- Les traitements d’import et de résolution sont idempotents.

## 3. Cycle de vie des lots

Les statuts autorisés sont :

- `PENDING` : lot créé mais pas encore traité ;
- `PROCESSING` : lecture, normalisation et comparaison en cours ;
- `COMPLETED` : traitement terminé sans anomalie ouverte ;
- `COMPLETED_WITH_MISMATCHES` : traitement terminé avec anomalie à examiner ;
- `FAILED` : traitement interrompu, avec motif exploitable mais sans secret.

Un lot expose les compteurs `totalItems`, `matchedItems`, `mismatchedItems`, `resolvedItems` et `ignoredItems`. Les compteurs doivent rester cohérents avec les éléments persistés.

## 4. Cycle de vie des éléments

Les statuts des éléments sont `PENDING`, `MATCHED`, `PARTIALLY_MATCHED`, `MISMATCHED`, `RESOLVED` et `IGNORED`.

Les causes minimales d’écart sont :

- transaction interne absente ;
- transaction partenaire absente ;
- montant différent ;
- devise différente ;
- statut différent ;
- transaction partenaire dupliquée ;
- autre cause explicitement documentée.

`MATCHED`, `RESOLVED` et `IGNORED` sont des états finaux. Une anomalie `MISMATCHED` ou `PARTIALLY_MATCHED` peut être résolue ou ignorée, mais uniquement avec une note non vide.

## 5. Contrat API

Le contrat partagé est défini dans `mansa-platform/packages/contracts/src/reconciliation-api.ts`.

Routes :

- `GET /v1/reconciliation/batches` : lister les lots avec pagination et filtres ;
- `GET /v1/reconciliation/batches/:batchId` : consulter un lot ;
- `GET /v1/reconciliation/items` : lister les éléments avec pagination et filtres ;
- `GET /v1/reconciliation/items/:itemId` : consulter un élément ;
- `POST /v1/reconciliation/items/:itemId/resolve` : résoudre ou ignorer une anomalie.

La commande de résolution contient obligatoirement :

- le statut cible `RESOLVED` ou `IGNORED` ;
- une note de résolution ;
- un code motif ;
- une clé d’idempotence ;
- un identifiant de corrélation ;
- la date de mise à jour.

## 6. Filtres et accès

Les lots sont filtrables par partenaire, statut et période. Les éléments sont filtrables par lot, partenaire, statut, cause d’écart, références interne et partenaire, et période de création.

Les accès doivent être limités au périmètre pays, organisation et partenaire de l’administrateur. Les exports et résolutions en masse exigent une autorisation distincte et, selon le niveau de risque, une double validation.

## 7. Traitement asynchrone

Le worker de réconciliation doit :

1. vérifier l’intégrité et le format de la source ;
2. normaliser les références, montants, devises, dates et statuts ;
3. dédupliquer les lignes partenaire ;
4. retrouver les opérations Mansa candidates ;
5. appliquer les règles de correspondance déterministes ;
6. enregistrer chaque résultat et mettre à jour les compteurs du lot ;
7. publier les métriques et alertes ;
8. clôturer le lot de manière atomique.

Un échec partiel doit permettre une reprise sans créer de doublons.

## 8. Contrôles et alertes

Une alerte est déclenchée lorsque :

- le taux ou le montant des écarts dépasse un seuil configurable ;
- un lot n’est pas terminé dans le délai prévu ;
- le nombre d’opérations dupliquées augmente anormalement ;
- un partenaire ne fournit pas son fichier attendu ;
- les compteurs du lot ne correspondent pas aux éléments persistés ;
- une résolution manuelle produit ou nécessite une correction comptable.

Les tableaux de bord présentent au minimum le volume traité, le taux de rapprochement, le montant total des écarts, l’ancienneté des anomalies et le délai moyen de résolution.

## 9. Critères d’acceptation

- Un lot sans écart se termine en `COMPLETED` avec tous ses éléments en `MATCHED`.
- Un lot contenant au moins une anomalie ouverte se termine en `COMPLETED_WITH_MISMATCHES`.
- La même source rejouée avec la même clé ne crée aucun doublon.
- Une résolution sans motif, note, corrélation ou idempotence est refusée.
- Un utilisateur hors périmètre ne peut ni lire ni résoudre l’anomalie.
- Une correction financière génère un journal compensatoire et conserve le lien avec l’élément de réconciliation.
- Les compteurs d’un lot sont recalculables à partir de ses éléments.
- Les journaux et métriques ne contiennent aucun secret, numéro de carte complet ou donnée KYC brute.

## 10. Travaux techniques restants

- persistance PostgreSQL des lots et éléments ;
- import sécurisé des formats CSV, JSON et formats partenaires ;
- règles de rapprochement configurables et versionnées ;
- worker idempotent et mécanisme de reprise ;
- intégration au grand livre et aux règlements ;
- écrans d’analyse, résolution et export dans le portail administrateur ;
- tests unitaires, d’intégration, de charge et de reprise.
