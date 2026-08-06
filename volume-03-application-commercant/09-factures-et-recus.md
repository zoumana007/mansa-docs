# Factures et reçus commerçant

## Objectif

Le module de facturation transforme une vente, une commande ou une prestation en document commercial traçable. Il doit fonctionner depuis l’application Commerçant, le TPE, le mini-site et les intégrations API sans dupliquer les données sensibles du paiement.

## Distinction entre reçu et facture

- Le reçu confirme un encaissement déjà réalisé.
- La facture décrit une créance, peut être émise avant paiement et peut rester partiellement ou totalement impayée.
- Une commande peut exister sans facture, mais une facture liée à une commande conserve sa référence.
- Un paiement peut régler une ou plusieurs factures selon les règles du backend ; le premier contrat partagé couvre le rattachement d’un paiement à une facture.

## Cycle de vie

Les statuts initiaux sont :

- `DRAFT` : document modifiable et non communiqué au client ;
- `ISSUED` : facture numérotée et émise ;
- `PARTIALLY_PAID` : une partie du montant a été encaissée ;
- `PAID` : solde intégralement réglé ;
- `OVERDUE` : échéance dépassée avec un solde restant ;
- `CANCELLED` : facture annulée avec motif ;
- `REFUNDED` : montant payé remboursé selon les règles de remboursement.

Une facture émise n’est pas supprimée. Les corrections sont réalisées par annulation, avoir ou document compensatoire selon les exigences réglementaires du pays.

## Numérotation

Le numéro de facture est généré côté serveur selon une séquence configurable par commerçant, pays, établissement et exercice. Il doit être unique dans son périmètre et ne peut pas être modifié après émission.

Le format peut contenir un préfixe non sensible, l’année et une séquence. L’absence temporaire de connexion ne doit jamais permettre à deux terminaux de produire le même numéro définitif ; un brouillon local reçoit un identifiant provisoire puis le backend attribue le numéro officiel.

## Parties et données client

La facture contient le nom d’affichage du client et, lorsque nécessaire, des coordonnées ou références fiscales. Les listes et journaux utilisent des valeurs masquées. Les documents générés ne doivent inclure que les données légalement nécessaires.

Aucun numéro de carte complet, code OTP, secret, document KYC ou donnée d’authentification ne doit apparaître dans une facture ou un reçu.

## Lignes et montants

Chaque ligne conserve un instantané de la désignation, du produit ou de la variante, du SKU, de la quantité, du prix unitaire, de la remise, de la taxe et du total de ligne.

La ventilation globale contient :

- sous-total ;
- total des remises ;
- total des taxes ;
- montant dû ;
- montant payé ;
- solde restant.

Tous les montants utilisent la devise de la facture et les unités mineures entières. Les calculs sont effectués et validés côté serveur.

## Émission et distribution

Les canaux initiaux sont `IN_APP`, `EMAIL`, `SMS`, `WHATSAPP` et `PRINT`. Une demande d’émission peut sélectionner plusieurs canaux.

Le service de notification reçoit uniquement les références et données nécessaires. Chaque livraison possède son propre identifiant, son statut et sa traçabilité. Les liens de téléchargement sont temporaires et ne contiennent aucun secret permanent.

## Paiement

L’enregistrement d’un paiement exige :

- l’identifiant de facture ;
- l’identifiant autoritatif du paiement ;
- le montant affecté ;
- l’acteur ;
- une clé d’idempotence.

Le backend vérifie la devise, le montant disponible, le solde restant et l’absence de double affectation. Le statut devient `PARTIALLY_PAID` ou `PAID` selon le nouveau solde.

## Retard et relance

Le statut `OVERDUE` est calculé par le backend lorsque l’échéance est dépassée et que le solde reste positif. Les relances éventuelles utilisent le module de notification, les préférences du client et une fréquence configurable.

Une relance ne doit jamais révéler des données financières détaillées sur un canal non authentifié au-delà de ce qui est strictement nécessaire.

## Annulation et remboursement

Une annulation exige un motif, un acteur autorisé et une clé d’idempotence. Une facture déjà payée ne peut pas être simplement supprimée ; elle passe par le processus de remboursement ou d’avoir.

Les remboursements restent rattachés au paiement et au grand livre. La facture conserve les références nécessaires à l’audit sans dupliquer les écritures comptables.

## API

Les routes initiales sont :

- `GET /v1/merchant/invoices` ;
- `POST /v1/merchant/invoices` ;
- `GET /v1/merchant/invoices/:invoiceId` ;
- `POST /v1/merchant/invoices/:invoiceId/issue` ;
- `POST /v1/merchant/invoices/:invoiceId/payments` ;
- `POST /v1/merchant/invoices/:invoiceId/cancel` ;
- `GET /v1/merchant/invoices/:invoiceId/document`.

Les listes sont paginées et filtrables par commerçant, point de vente, client, commande, statut et période.

## Sécurité et audit

- Les accès sont limités au commerçant et aux établissements autorisés.
- La création, l’émission, l’annulation, l’affectation de paiement et le téléchargement sont audités.
- Les liens de document expirent et sont générés à la demande.
- Les documents sont stockés dans un service sécurisé avec politique de rétention.
- Les modèles de facture sont configurables, mais les mentions obligatoires ne peuvent pas être supprimées sans contrôle administratif.

## Contrat technique

Le domaine partagé est défini dans `mansa-platform/packages/contracts/src/invoice.ts` et exposé par `@mansa/contracts/invoice`.

Le contrat d’API est défini dans `mansa-platform/packages/contracts/src/invoice-api.ts` et exposé par `@mansa/contracts/invoice-api`.

## Critères d’acceptation

1. Deux créations avec la même clé d’idempotence ne produisent qu’une facture.
2. Une facture émise reçoit un numéro unique et immuable.
3. Tous les montants utilisent la devise de la facture.
4. Un paiement déjà affecté n’est pas comptabilisé deux fois.
5. Le solde restant ne devient jamais négatif.
6. Une facture partiellement réglée passe à `PARTIALLY_PAID`.
7. Une facture totalement réglée passe à `PAID`.
8. Une facture échue avec solde positif passe à `OVERDUE`.
9. Une facture émise n’est jamais supprimée physiquement par une action métier.
10. Les documents et journaux n’exposent aucun secret ni donnée de carte complète.
