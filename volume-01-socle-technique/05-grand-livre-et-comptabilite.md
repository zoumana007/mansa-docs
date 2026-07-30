# Volume 1 — Grand livre et comptabilité

## 1. Rôle du grand livre

Le grand livre est la source de vérité financière de Mansa. Aucun solde affiché dans une application ne doit être modifié directement. Toute variation de valeur provient d’une écriture équilibrée, immuable et rattachée à une transaction métier.

## 2. Partie double

Chaque opération produit au minimum deux lignes :

- une ligne au débit d’un compte ;
- une ligne au crédit d’un autre compte ;
- un total débit strictement égal au total crédit pour une même devise.

Une écriture déséquilibrée est rejetée avant persistance.

## 3. Types de comptes

- Actif client disponible.
- Actif client réservé.
- Dette envers client.
- Compte commerçant.
- Compte de règlement partenaire.
- Compte de commissions Mansa.
- Compte de taxes et prélèvements.
- Compte de compensation.
- Compte technique d’attente.

Les comptes techniques doivent être suivis et rapprochés quotidiennement. Ils ne servent jamais à masquer une anomalie.

## 4. Structure minimale d’une écriture

- Identifiant unique.
- Identifiant de transaction métier.
- Identifiant d’idempotence.
- Compte débité.
- Compte crédité.
- Montant en unité mineure.
- Devise ISO 4217.
- Nature de l’opération.
- Référence partenaire éventuelle.
- Horodatage métier et horodatage d’enregistrement.
- Acteur ou système initiateur.
- Métadonnées non sensibles.

## 5. États

Une transaction financière suit un cycle explicite : `CREATED`, `PENDING`, `AUTHORIZED`, `SETTLED`, `FAILED`, `CANCELLED` ou `REVERSED`.

Le passage à `SETTLED` confirme le règlement final. Une correction après règlement se fait par contre-écriture liée à l’écriture initiale, jamais par modification ou suppression.

## 6. Réservations

Les paiements par carte, TPE, transfert différé ou partenaire asynchrone peuvent réserver des fonds avant règlement. Une réservation possède une date d’expiration, un propriétaire, un montant, une devise et un statut. À l’expiration, elle est libérée de manière idempotente.

## 7. Solde

Le solde disponible est calculé à partir des écritures comptables validées moins les réservations actives. Des vues matérialisées ou agrégats de performance peuvent être utilisés, mais ils restent reconstruisibles depuis le journal.

## 8. Rapprochement

Chaque partenaire externe fournit des fichiers, API ou événements de règlement. Mansa compare quotidiennement :

- les transactions internes ;
- les références partenaires ;
- les montants et devises ;
- les dates de valeur ;
- les commissions ;
- les rejets et annulations.

Tout écart ouvre une anomalie traçable avec propriétaire, priorité et résolution.

## 9. Contrôles obligatoires

- Montant strictement positif.
- Devise identique sur toutes les lignes d’une écriture.
- Comptes autorisés pour la nature d’opération.
- Somme des débits égale à la somme des crédits.
- Idempotence vérifiée avant création.
- Interdiction de supprimer une écriture comptable.
- Référence obligatoire pour toute contre-écriture.
- Journal d’audit séparé pour les actions administratives.

## 10. Critères de validation

- Tests unitaires sur équilibre, devise et montants.
- Tests de concurrence sur réservations et règlements.
- Reconstruction complète d’un solde depuis les écritures.
- Rapprochement d’un jeu de données de référence.
- Vérification qu’aucune route d’administration ne modifie directement un solde.
