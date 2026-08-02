# Règlement des commerçants

## Objet

Le règlement transforme les encaissements éligibles d’un commerçant en un lot payable vers un compte bancaire, un compte Mobile Money ou un portefeuille Mansa. Il intervient après la comptabilisation, les contrôles de risque et le rapprochement des opérations concernées.

## Principes

- Un lot appartient à un seul commerçant, une seule devise et une période déterminée.
- Tous les montants sont exprimés dans l’unité monétaire mineure avec des entiers sûrs.
- Le montant net est calculé par `montant brut - frais + ajustements`.
- Un lot ne modifie jamais directement le grand livre : il référence les écritures qui justifient son montant.
- Une destination de paiement est une référence tokenisée ou interne, jamais un secret bancaire en clair.
- Les traitements sont idempotents et utilisent une référence partenaire unique.
- Les échecs peuvent être repris sans créer un second paiement.
- Les règlements importants ou exceptionnels peuvent nécessiter une validation à quatre yeux.

## Destinations

- `BANK_ACCOUNT` : compte bancaire validé du commerçant ;
- `MOBILE_MONEY` : compte opérateur vérifié ;
- `MANSA_WALLET` : portefeuille Mansa appartenant au commerçant.

La référence de destination doit désigner un moyen préalablement vérifié. Les coordonnées sensibles complètes restent dans le coffre de secrets ou le système partenaire adapté.

## États du lot

- `DRAFT` : lot calculé mais encore modifiable ;
- `READY` : contrôles terminés et lot prêt à être envoyé ;
- `PROCESSING` : instruction transmise au partenaire ;
- `PAID` : paiement confirmé intégralement ;
- `PARTIALLY_PAID` : paiement partiel nécessitant une reprise ou une analyse ;
- `FAILED` : échec confirmé avec motif ;
- `CANCELLED` : lot annulé avant paiement.

Les états finaux sont `PAID` et `CANCELLED`.

## Transitions autorisées

1. `DRAFT` vers `READY` ou `CANCELLED` ;
2. `READY` vers `PROCESSING` ou `CANCELLED` ;
3. `PROCESSING` vers `PAID`, `PARTIALLY_PAID` ou `FAILED` ;
4. `PARTIALLY_PAID` vers `PROCESSING`, `PAID` ou `FAILED` ;
5. `FAILED` vers `READY` ou `CANCELLED`.

Une transition hors de ce cycle est rejetée. Un lot payé ou annulé ne peut plus être rouvert automatiquement.

## Calcul du montant

Le lot expose au minimum :

- montant brut des opérations éligibles ;
- frais et commissions retenus ;
- ajustements signés ;
- montant net ;
- devise ;
- période de début et de fin.

Un montant brut ou des frais négatifs sont interdits. Les ajustements peuvent être positifs ou négatifs, mais le montant net final ne peut pas être négatif.

## Traitement opérationnel

Le traitement planifié doit :

1. sélectionner les opérations éligibles et non déjà réglées ;
2. figer la période et les références comptables ;
3. calculer le brut, les frais, les ajustements et le net ;
4. vérifier la destination et les limites du commerçant ;
5. soumettre les lots nécessitant une approbation ;
6. transmettre l’instruction au partenaire ;
7. enregistrer la référence partenaire ;
8. rapprocher la confirmation de paiement ;
9. notifier le commerçant et produire son relevé.

Les lots doivent pouvoir être recalculés en mode simulation avant leur passage à `READY`.

## Échecs et reprises

Un passage à `FAILED` exige un motif non vide. Les erreurs doivent être classées au minimum entre indisponibilité temporaire, destination invalide, rejet conformité, fonds indisponibles chez le partenaire et erreur inconnue.

Une reprise repart de `FAILED` vers `READY`, conserve l’identité du lot et génère une nouvelle tentative technique sans dupliquer le règlement métier.

## Sécurité et audit

Les événements suivants sont audités : création, recalcul, changement de destination, approbation, transmission, confirmation, échec, reprise et annulation.

Les rôles de préparation, approbation et exécution doivent pouvoir être séparés. Une modification de destination proche d’un règlement déclenche un contrôle renforcé et, selon la politique, une période de refroidissement.

## Observabilité

Les métriques minimales sont :

- nombre et montant des lots par état, devise et partenaire ;
- délai moyen entre fin de période et paiement ;
- taux d’échec et de reprise ;
- montants partiellement payés ;
- règlements bloqués par conformité ou approbation ;
- écarts entre lot, grand livre et confirmation partenaire.

## Alignement avec le code

Le contrat de référence est `packages/contracts/src/settlement.ts` dans `mansa-platform`.

Il expose :

- `SETTLEMENT_BATCH_STATUSES` et `SETTLEMENT_DESTINATION_TYPES` ;
- `SettlementBatch`, `SettlementDestination` et les commandes associées ;
- `createSettlementBatch` ;
- `transitionSettlementBatch` ;
- `canTransitionSettlementBatch` ;
- `isFinalSettlementBatchStatus` ;
- le sous-chemin public `@mansa/contracts/settlement`.

## Critères d’acceptation

- La période de début précède strictement la période de fin.
- Le brut et les frais sont des entiers sûrs non négatifs.
- Le calcul du montant net est déterministe et ne produit jamais un montant négatif.
- La devise est normalisée sur trois lettres.
- Une destination sans référence est rejetée.
- Seules les transitions prévues sont acceptées.
- `PAID` et `PARTIALLY_PAID` exigent une référence partenaire.
- `FAILED` exige un motif explicite.
- Une reprise après échec conserve l’identifiant du lot.
- Les tests couvrent le calcul, le cycle nominal, les échecs, les reprises et les entrées invalides.
- La documentation et les exports publics restent synchronisés.
