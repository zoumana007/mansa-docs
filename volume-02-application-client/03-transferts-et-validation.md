# Transferts, devis et validation forte

## Objectif

Le parcours de transfert permet au client d’envoyer de l’argent vers un autre portefeuille Mansa, un compte bancaire, un compte Mobile Money, un commerçant ou un service public. Le traitement doit rester idempotent, traçable et compatible avec les contrôles de conformité.

## Types de transfert

- `INTERNAL` : portefeuille Mansa vers portefeuille Mansa.
- `BANK` : transfert vers un compte bancaire partenaire.
- `MOBILE_MONEY` : transfert vers un opérateur Mobile Money.
- `MERCHANT` : paiement ou transfert vers un commerçant enregistré.
- `PUBLIC_SERVICE` : règlement d’une obligation ou d’un service public.

## Parcours fonctionnel

1. Le client sélectionne un bénéficiaire actif.
2. Il saisit le montant et une référence facultative.
3. L’application demande un devis au backend.
4. Le devis retourne le montant, les frais de service, les frais partenaire, la taxe, le total débité et une date d’expiration.
5. Le client confirme le récapitulatif.
6. Le backend crée le transfert avec une clé d’idempotence.
7. Une authentification forte est demandée selon le risque : PIN, biométrie, OTP ou authentification renforcée.
8. Le transfert passe dans le traitement partenaire ou interne.
9. L’application affiche le résultat et produit un reçu.

## États

- `CREATED`
- `PENDING_AUTHORIZATION`
- `PENDING`
- `PROCESSING`
- `COMPLETED`
- `FAILED`
- `CANCELLED`
- `REVERSED`

Les transitions terminales sont `COMPLETED`, `FAILED`, `CANCELLED` et `REVERSED`. Une annulation client n’est permise que tant que le transfert n’est pas irrévocablement engagé auprès du partenaire.

## Codes d’échec normalisés

- `INSUFFICIENT_FUNDS`
- `LIMIT_EXCEEDED`
- `BENEFICIARY_BLOCKED`
- `COMPLIANCE_REVIEW_REQUIRED`
- `PROVIDER_UNAVAILABLE`
- `DESTINATION_REJECTED`
- `TECHNICAL_ERROR`

Les messages présentés au client doivent être compréhensibles sans exposer les détails internes, les règles antifraude ou les données du partenaire.

## Règles métier

- Les montants utilisent le type partagé `Money` et jamais un nombre flottant.
- La devise du portefeuille source, du devis et des frais doit être cohérente.
- Le devis expire et ne peut pas être réutilisé après son expiration.
- Une clé d’idempotence identique ne doit jamais créer deux transferts.
- Le bénéficiaire doit être actif et autorisé pour le client.
- Les limites KYC, journalières, mensuelles, pays, partenaire et produit sont contrôlées avant autorisation.
- Une décision de conformité peut placer l’opération en revue sans la déclarer réussie.
- Toute transition d’état produit un événement d’audit.
- Les informations sensibles de destination restent masquées dans les réponses destinées au mobile.

## Contrats techniques alignés

Le dépôt `mansa-platform` expose dans `packages/contracts/src/transfer.ts` les catalogues `TRANSFER_TYPES`, `TRANSFER_STATUSES`, `TRANSFER_FAILURE_CODES`, les gardes de type associées, les commandes de devis, création, autorisation et annulation, ainsi que les objets `TransferQuote`, `Transfer` et `TransferFeeBreakdown`.

## Critères d’acceptation

1. Un double appui sur le bouton de confirmation ne crée qu’un seul transfert.
2. Un devis expiré est refusé et l’application demande un nouveau devis.
3. Un bénéficiaire bloqué ne peut pas recevoir de transfert.
4. Les frais affichés avant confirmation sont identiques aux frais comptabilisés.
5. Un échec partenaire conserve une trace exploitable et n’est jamais affiché comme un succès.
6. Une opération terminée possède un reçu et une référence consultable dans l’historique.
7. Les tests de contrats vérifient les valeurs autorisées et l’absence de doublons dans les catalogues.
