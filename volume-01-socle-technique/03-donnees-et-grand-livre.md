# Volume 1 — Données et grand livre

## 1. Règles de représentation

Tous les montants sont stockés en unités mineures avec des entiers signés. Une valeur `1250` en XOF représente 1 250 FCFA, la devise n’ayant pas de décimales. Chaque montant transporte explicitement son code devise ISO 4217.

Les identifiants sont opaques, non séquentiels et générés côté serveur. Les dates sont conservées en UTC et rendues dans le fuseau de l’utilisateur uniquement à l’affichage.

## 2. Grand livre en partie double

Toute opération financière validée produit une écriture équilibrée composée d’au moins deux lignes :

- un débit sur un compte comptable ;
- un crédit de même montant et de même devise sur un autre compte.

La somme algébrique des lignes d’une écriture doit toujours être nulle. Une écriture publiée n’est jamais modifiée ni supprimée. Toute correction utilise une contre-écriture liée à l’écriture d’origine.

## 3. Entités principales

- `Customer` : client personne physique ou morale.
- `Wallet` : portefeuille fonctionnel visible par l’utilisateur.
- `LedgerAccount` : compte comptable interne.
- `JournalEntry` : opération comptable atomique.
- `JournalLine` : ligne de débit ou de crédit.
- `PaymentIntent` : intention de paiement avant règlement définitif.
- `Transfer` : mouvement entre portefeuilles ou partenaires.
- `ExternalTransaction` : référence d’une opération chez un partenaire.
- `IdempotencyKey` : protection contre les doubles traitements.
- `AuditEvent` : preuve d’une action humaine ou système.

## 4. États transactionnels

Une transaction suit une machine à états explicite :

`CREATED -> PENDING -> AUTHORIZED -> SETTLED`

Des branches contrôlées permettent `FAILED`, `CANCELLED`, `EXPIRED`, `REVERSED` et `DISPUTED`. Toute transition enregistre l’acteur, la cause, l’horodatage et les références externes.

## 5. Idempotence

Toute commande financière exige une clé d’idempotence unique par acteur et type d’opération. La première exécution conserve :

- l’empreinte de la requête ;
- le statut de traitement ;
- la réponse finale ;
- la durée de conservation.

Une réutilisation avec un contenu différent est rejetée. Une réutilisation identique retourne la réponse initiale sans créer une nouvelle transaction.

## 6. Cohérence et concurrence

- Les écritures du grand livre sont créées dans une transaction PostgreSQL atomique.
- Les soldes disponibles ne sont jamais calculés à partir d’un cache seul.
- Les verrous sont courts et ciblés.
- Les traitements externes utilisent une boîte d’envoi transactionnelle afin d’éviter la perte d’événements.
- Les consommateurs d’événements sont eux-mêmes idempotents.

## 7. Conservation et confidentialité

Les durées de conservation sont configurées par catégorie de données et juridiction. Les documents KYC et données sensibles utilisent un stockage chiffré, des accès temporaires signés et une journalisation des consultations. Les données utilisées pour les tests sont synthétiques ou anonymisées.

## 8. Critères d’acceptation

- Impossible de publier une écriture déséquilibrée.
- Impossible de mélanger plusieurs devises dans une même écriture.
- Impossible de modifier une écriture publiée.
- Une commande répétée avec la même clé ne produit qu’un seul effet financier.
- Chaque solde affiché peut être rapproché avec les lignes comptables.
