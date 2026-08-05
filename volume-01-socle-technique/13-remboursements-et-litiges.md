# Remboursements et litiges

## Objectif

Ce domaine couvre deux traitements distincts : le remboursement initié par Mansa ou un commerçant et le litige transmis par un client, une banque, un réseau de cartes ou un partenaire. Ils doivent rester corrélés au paiement d’origine, au grand livre, aux règlements marchands et aux preuves disponibles.

## Remboursements

Les statuts sont `REQUESTED`, `REVIEW_REQUIRED`, `APPROVED`, `PROCESSING`, `SUCCEEDED`, `FAILED` et `CANCELLED`.

Règles principales :

- le montant est un entier positif en unités mineures ;
- la devise doit correspondre à celle du paiement d’origine ;
- la somme des remboursements réussis ne peut pas dépasser le montant capturé ;
- toute création utilise une clé d’idempotence et un identifiant de corrélation ;
- les remboursements dépassant un seuil configurable peuvent exiger une revue ou une double validation ;
- un échec doit contenir un code explicite ;
- seuls les statuts autorisés par la machine d’état peuvent être appliqués ;
- un remboursement réussi produit les écritures comptables inverses appropriées et ajuste le prochain règlement marchand si nécessaire.

Routes initiales :

- `GET /v1/refunds` ;
- `POST /v1/refunds` ;
- `GET /v1/refunds/:refundId` ;
- `POST /v1/refunds/:refundId/status`.

## Litiges

Les statuts sont `OPENED`, `EVIDENCE_REQUIRED`, `UNDER_REVIEW`, `WON`, `LOST`, `WITHDRAWN` et `EXPIRED`.

Règles principales :

- chaque litige référence un paiement et une échéance de réponse ;
- une décision `WON` ou `LOST` exige une note de résolution ;
- le passage en revue exige au moins une preuve ;
- les preuves ne contiennent dans les contrats qu’une référence de stockage sécurisée, jamais le fichier ou un secret ;
- aucune preuve ne peut être ajoutée après un statut final ;
- les litiges en retard doivent être détectables et priorisés ;
- la perte d’un litige déclenche les écritures, frais et ajustements de règlement prévus par le partenaire ;
- les accès aux preuves sont limités, journalisés et soumis aux règles de conservation.

Routes initiales :

- `GET /v1/disputes` ;
- `POST /v1/disputes` ;
- `GET /v1/disputes/:disputeId` ;
- `POST /v1/disputes/:disputeId/evidence` ;
- `POST /v1/disputes/:disputeId/status`.

## Contrats techniques

Les domaines sont définis dans `packages/contracts/src/refund.ts` et `packages/contracts/src/dispute.ts`. Les contrats de transport sont définis dans `packages/contracts/src/refund-api.ts` et `packages/contracts/src/dispute-api.ts`, puis réexportés par `packages/contracts/src/api-contracts.ts`.

## Audit, sécurité et confidentialité

Toute création, transition, ajout de preuve, décision et reprise est auditée avec acteur, portée, résultat, corrélation et horodatage. Les numéros de carte, documents KYC, coordonnées et références sensibles doivent être masqués. Les pièces justificatives sont analysées avant stockage, chiffrées et accessibles par URL temporaire ou identifiant opaque.

## Critères d’acceptation

- une même clé d’idempotence ne crée pas deux remboursements ou deux litiges ;
- un remboursement supérieur au montant encore remboursable est rejeté ;
- une transition interdite ne modifie aucun état ;
- un remboursement échoué sans code d’échec est refusé ;
- un litige ne passe pas en revue sans preuve ;
- une décision finale sans note de résolution est refusée ;
- une preuve ajoutée à un litige final est rejetée ;
- chaque événement reste traçable jusqu’au paiement, aux écritures de ledger, au règlement marchand et au partenaire concerné.

## Suite

Le prochain lot doit ajouter la persistance, les modules NestJS, les workers partenaires, les permissions détaillées, les tests contractuels, les écritures comptables automatiques et les écrans client, commerçant et administration.
