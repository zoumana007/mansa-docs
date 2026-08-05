# Rapprochement financier

## 1. Objectif

Le rapprochement financier compare les opérations internes de Mansa avec les relevés des banques, opérateurs Mobile Money, processeurs cartes et autres partenaires. Il doit détecter automatiquement les écarts, permettre leur traitement contrôlé et conserver une piste d’audit complète.

## 2. Principes

- Un rapprochement est exécuté par partenaire et par période.
- Une ligne peut être rapprochée, partiellement rapprochée, en anomalie, résolue ou ignorée.
- Aucun écart ne doit être supprimé pour faire disparaître une anomalie.
- Toute résolution manuelle exige un motif, un acteur, une date, un identifiant de corrélation et une clé d’idempotence.
- Les montants sont exprimés en unité monétaire mineure et ne doivent jamais utiliser de nombre flottant.
- Les fichiers partenaires sont stockés hors du dépôt, chiffrés et référencés par un identifiant technique.

## 3. Motifs d’écart

Le socle prend en charge au minimum :

- transaction interne absente ;
- transaction partenaire absente ;
- montant différent ;
- devise différente ;
- statut différent ;
- doublon côté partenaire ;
- anomalie non classée.

## 4. Cycle de traitement

1. Import sécurisé du relevé partenaire.
2. Validation du format, de la période, du partenaire et de l’intégrité du fichier.
3. Normalisation des références et montants.
4. Correspondance automatique avec les transactions internes.
5. Classement des résultats et production des statistiques du lot.
6. Analyse des anomalies par un rôle autorisé.
7. Résolution ou classement sans suite avec justification.
8. Export du rapport et archivage des preuves.

## 5. Contrats API

Les contrats sont définis dans `packages/contracts/src/reconciliation.ts` et `packages/contracts/src/reconciliation-api.ts`.

Routes initiales :

- `GET /v1/reconciliation/batches` ;
- `GET /v1/reconciliation/batches/:batchId` ;
- `GET /v1/reconciliation/items` ;
- `GET /v1/reconciliation/items/:itemId` ;
- `POST /v1/reconciliation/items/:itemId/resolve`.

Les listes sont paginées par curseur. Les filtres couvrent le partenaire, le lot, le statut, le motif d’écart et la période de création.

## 6. Autorisations

- Un analyste financier peut consulter les lots et anomalies de son périmètre.
- Un responsable financier peut résoudre un écart dans les limites définies.
- Les écarts au-dessus d’un seuil configurable nécessitent une double validation.
- Un administrateur technique ne peut pas modifier une décision financière sans autorisation métier explicite.
- Les actions de résolution et d’ignorance sont toujours auditées.

## 7. Contrôles de sécurité

- Vérifier la signature, le hash ou le canal sécurisé du fichier partenaire.
- Rejeter les fichiers déjà importés sauf reprise explicitement autorisée.
- Masquer les références sensibles dans les journaux applicatifs.
- Interdire l’exposition d’identifiants bancaires complets dans les exports standards.
- Conserver les preuves conformément à la politique de rétention applicable.

## 8. Critères d’acceptation

- Un lot entièrement concordant est clôturé sans anomalie.
- Une opération absente d’un côté est classée avec le motif approprié.
- Une différence de montant est détectée même si les références correspondent.
- Une résolution sans note est refusée.
- Une seconde soumission avec la même clé d’idempotence ne crée pas une nouvelle décision.
- Les statistiques d’un lot correspondent au nombre réel de lignes par statut.
- Les consultations et décisions respectent le périmètre RBAC du demandeur.

## 9. Éléments restant à implémenter

- ingestion des formats CSV, ISO 20022 et formats propriétaires ;
- moteur de correspondance configurable ;
- persistance des lots et anomalies ;
- workflow de double validation ;
- écrans du portail administrateur ;
- exports comptables et rapports signés ;
- alertes et indicateurs d’ancienneté des anomalies.
