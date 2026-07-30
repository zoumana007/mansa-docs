# Volume 2 — Paiements, QR et demandes d’argent

## 1. Objectif

L’application Client doit permettre de payer un commerçant, une personne ou un service public avec un parcours court, compréhensible et traçable. Les canaux initiaux sont le QR statique, le QR dynamique, les demandes de paiement et les paiements initiés par un commerçant ou un TPE.

## 2. Principes métier

- Tout paiement est créé avec une clé d’idempotence.
- Le montant est exprimé en unités mineures et lié à une devise ISO 4217.
- Le client voit le bénéficiaire, le montant, les frais et le total avant confirmation.
- Aucun débit définitif ne peut être effectué sans validation des limites, du solde, du statut du compte et du niveau KYC.
- Les statuts sont explicites et ne sont jamais déduits uniquement d’un message partenaire.
- Toute annulation, expiration ou inversion produit une trace d’audit et, si nécessaire, une écriture compensatoire.

## 3. Types de QR

### QR statique

Le QR identifie le bénéficiaire. Le payeur saisit le montant et éventuellement une référence. Il convient aux petits commerces et paiements simples.

### QR dynamique

Le QR contient une intention de paiement signée avec : bénéficiaire, montant, devise, référence, date d’expiration et contexte. Il convient aux caisses, factures, restaurants, administrations et TPE.

### QR personnel

Chaque client peut afficher un QR de réception. Le QR ne contient aucune donnée sensible brute et peut être régénéré ou désactivé.

## 4. Cycle de vie d’un paiement

1. Création de l’intention.
2. Vérification de validité, du bénéficiaire et de l’expiration.
3. Calcul des frais et taxes.
4. Présentation du récapitulatif.
5. Authentification renforcée selon le risque.
6. Réservation ou débit atomique.
7. Confirmation du canal ou partenaire externe.
8. Finalisation, notification et reçu.
9. Rapprochement asynchrone lorsque le partenaire ne répond pas immédiatement.

Statuts de référence : `CREATED`, `REQUIRES_ACTION`, `PROCESSING`, `SUCCEEDED`, `FAILED`, `CANCELLED`, `EXPIRED`, `REVERSED`.

## 5. Demandes d’argent

Un utilisateur peut créer une demande adressée à un autre utilisateur ou partageable par lien/QR. La demande possède un montant, une devise, une description, une expiration et un statut. Le destinataire peut accepter, refuser ou ignorer la demande. L’acceptation crée un paiement séparé afin de conserver l’idempotence et la traçabilité.

## 6. Paiement partagé

Une facture peut être divisée :

- à parts égales ;
- par montants personnalisés ;
- par articles attribués à chaque participant.

Le commerçant ne voit que l’état global et les parts payées, sans accéder aux informations financières privées des participants. Une part impayée ne doit pas modifier les paiements déjà finalisés.

## 7. Sécurité

- Signature des QR dynamiques.
- Expiration courte des intentions sensibles.
- Protection anti-rejeu par identifiant unique et nonce.
- Vérification du bénéficiaire après lecture du QR.
- Limites par utilisateur, appareil, commerce, pays et période.
- Authentification forte configurable selon montant, risque et nouveauté du bénéficiaire.
- Blocage immédiat possible depuis l’administration.

## 8. Reçus et contestations

Le reçu contient : identifiant, date, payeur masqué, bénéficiaire, montant, frais, total, canal, référence et statut. Un paiement réussi peut être signalé ou contesté selon les règles du produit, sans suppression de l’écriture originale.

## 9. Critères d’acceptation

- Un même appel avec la même clé d’idempotence ne crée jamais deux paiements.
- Un QR expiré ou falsifié est refusé.
- Les frais affichés avant confirmation correspondent aux frais comptabilisés.
- Les statuts finaux ne peuvent pas revenir vers un statut intermédiaire.
- Une inversion génère des écritures compensatoires et un lien vers l’opération d’origine.
- Les reçus sont consultables hors ligne après synchronisation.
