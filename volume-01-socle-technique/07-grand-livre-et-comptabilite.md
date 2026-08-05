# Grand livre et comptabilité en partie double

## 1. Objectif

Le grand livre constitue la source de vérité financière de Mansa. Tous les mouvements ayant un effet monétaire doivent produire une transaction comptable équilibrée, immuable et traçable avant d’être considérés comme définitivement comptabilisés.

Le solde affiché par une application ne doit jamais être calculé uniquement à partir d’un état mutable de portefeuille. Il doit être dérivable des écritures du grand livre, complété au besoin par des projections optimisées et recalculables.

## 2. Principes obligatoires

- Chaque transaction comporte au moins une écriture au débit et une écriture au crédit.
- La somme des débits doit être strictement égale à la somme des crédits.
- Une transaction ne mélange jamais plusieurs devises dans le même ensemble d’écritures.
- Les montants sont exprimés en unités mineures entières.
- Une écriture comptabilisée n’est jamais modifiée ni supprimée.
- Une correction s’effectue par une transaction d’extourne liée à la transaction d’origine.
- Chaque commande d’écriture possède une clé d’idempotence.
- Chaque transaction possède une référence métier, un identifiant de corrélation, un pays, une date d’événement et une source.
- Toute tentative non équilibrée ou incohérente est rejetée avant persistance.

## 3. Modèle de comptes

Les comptes utilisent les catégories comptables suivantes :

- `ASSET` : actifs détenus ou contrôlés par la plateforme ;
- `LIABILITY` : dettes de la plateforme envers clients, commerçants ou partenaires ;
- `EQUITY` : capitaux propres et comptes de clôture ;
- `REVENUE` : commissions et autres produits ;
- `EXPENSE` : frais, pertes et charges.

Un compte est rattaché à un propriétaire logique : plateforme, utilisateur, commerçant, partenaire ou organisme public. Les comptes système doivent être identifiables explicitement et ne peuvent pas être créés depuis une application cliente.

Chaque compte possède une devise unique et un pays de rattachement. Les transferts multidevises sont représentés par plusieurs transactions comptables reliées à une opération de change, et non par une seule transaction mélangeant plusieurs devises.

## 4. Cycle d’une transaction comptable

1. Le module métier prépare une commande d’écriture avec sa clé d’idempotence.
2. Le service vérifie le format, la devise, les comptes, les montants et l’équilibre.
3. Le système réserve les ressources nécessaires et empêche les doubles traitements concurrents.
4. La transaction est enregistrée avec le statut `PENDING`.
5. Toutes les écritures sont persistées atomiquement.
6. La transaction passe à `POSTED` et publie un événement transactionnel via l’outbox.
7. Les projections de soldes, historiques, notifications et analytics sont mises à jour de manière idempotente.

Une transaction rejetée passe à `REJECTED` sans écriture comptabilisée. Une transaction déjà `POSTED` ne peut être annulée qu’au moyen d’une extourne.

## 5. Extournes

L’extourne crée une nouvelle transaction contenant les écritures inverses de la transaction d’origine. Elle doit :

- référencer la transaction originale ;
- posséder sa propre clé d’idempotence ;
- conserver le motif et le code de raison ;
- utiliser le même pays et la même devise ;
- être auditée ;
- empêcher une double extourne non autorisée.

La transaction d’origine reste immuable et reçoit uniquement une relation vers la transaction d’extourne dans les projections de lecture.

## 6. Soldes

Le modèle distingue au minimum :

- solde comptable ;
- solde disponible ;
- montant en attente ou réservé ;
- date de calcul du solde.

Les autorisations de paiement, cautions et opérations asynchrones utilisent des réservations séparées. Une réservation ne devient une écriture définitive qu’au moment de la capture ou du dénouement métier.

Les projections de solde doivent pouvoir être reconstruites à partir des écritures. Une différence entre projection et grand livre déclenche une alerte de rapprochement et bloque, selon sa gravité, les nouvelles opérations du compte concerné.

## 7. API interne

Les contrats partagés sont définis dans :

- `packages/contracts/src/ledger.ts` ;
- `packages/contracts/src/ledger-api.ts`.

Les routes internes prévues couvrent :

- `POST /v1/internal/ledger/transactions` ;
- `GET /v1/internal/ledger/transactions/:transactionId` ;
- `POST /v1/internal/ledger/transactions/:transactionId/reverse` ;
- `GET /v1/internal/ledger/accounts/:accountId` ;
- `GET /v1/internal/ledger/accounts/:accountId/balance` ;
- `GET /v1/internal/ledger/accounts/:accountId/entries`.

Ces routes ne sont jamais exposées directement aux applications mobiles ou web. Les modules métier appellent le service de grand livre avec une identité de service authentifiée et un périmètre minimal.

## 8. Sécurité et audit

- Le service de grand livre refuse les appels utilisateurs directs.
- Les permissions d’écriture sont accordées uniquement aux services métier autorisés.
- Les créations de comptes système et extournes sensibles nécessitent une double validation administrative.
- Les journaux ne contiennent aucun numéro de carte complet, secret, code OTP ou document KYC.
- Les métadonnées sont limitées à un dictionnaire de chaînes validé et ne doivent pas recevoir de données personnelles libres.
- Les identifiants de corrélation relient transaction métier, événement, webhook, journal d’audit et trace distribuée.

## 9. Rapprochement

Le rapprochement compare le grand livre Mansa avec les relevés des banques, opérateurs Mobile Money, processeurs cartes et partenaires publics. Chaque écart est classé, assigné et résolu sans modifier les écritures historiques.

Les contrôles minimaux sont :

- totalité des transactions partenaires importées ;
- absence de doublons ;
- égalité des montants et devises ;
- cohérence des statuts ;
- cohérence entre règlements et comptes d’attente ;
- détection des transactions orphelines ;
- génération d’un rapport signé et archivé.

## 10. Critères d’acceptation

- Une transaction équilibrée est comptabilisée atomiquement.
- Une transaction déséquilibrée est rejetée sans écriture partielle.
- Deux commandes avec la même clé d’idempotence produisent une seule transaction logique.
- Une transaction multidevise est rejetée.
- Une extourne crée des écritures inverses et conserve la transaction originale.
- Une panne après persistance mais avant publication ne perd pas l’événement grâce à l’outbox.
- Une projection de solde peut être reconstruite et comparée au grand livre.
- Les routes internes refusent une identité utilisateur ou un service non autorisé.
- Les tests couvrent équilibre, concurrence, idempotence, extourne, reprise et rapprochement.

## 11. Éléments restant à construire

- persistance PostgreSQL transactionnelle ;
- tables de comptes, transactions, écritures, réservations et outbox ;
- module NestJS interne ;
- mécanisme de verrouillage et contrôle de concurrence ;
- projections de soldes ;
- workers de rapprochement ;
- tests de propriétés comptables et tests de charge ;
- runbooks d’incident financier et de reconstruction des projections.
