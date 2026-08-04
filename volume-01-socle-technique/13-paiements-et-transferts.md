# Paiements, demandes de paiement et transferts

## 1. Objectif

Ce document définit l’orchestration commune des paiements et transferts Mansa. Les contrats TypeScript correspondants se trouvent dans `@mansa/contracts`, notamment `payment.ts`, `payment-api.ts`, `payment-request.ts`, `transfer.ts` et `transfer-api.ts`.

Le module ne modifie jamais directement un solde. Toute réussite financière produit une transaction équilibrée dans le grand livre décrit dans `12-grand-livre-et-comptabilite.md`.

## 2. Principes obligatoires

- Toute création financière exige une clé d’idempotence.
- Les montants sont exprimés en unités mineures entières avec une devise explicite.
- Les frais sont calculés avant autorisation et conservés avec l’opération.
- Une opération n’est considérée réussie qu’après confirmation du fournisseur et publication comptable.
- Les appels partenaires sont isolés derrière des adaptateurs.
- Les tentatives techniques sont distinctes de l’opération métier afin de permettre les reprises sans doublon.
- Toute transition d’état est validée, horodatée et auditée.
- Les données sensibles de carte, secrets Mobile Money, codes PIN et OTP ne sont jamais journalisés.

## 3. Canaux de paiement

Les canaux normalisés sont :

- `INTERNAL` : paiement entre portefeuilles Mansa ;
- `QR_STATIC` : QR associé à un bénéficiaire, montant saisi par le payeur ;
- `QR_DYNAMIC` : QR contenant une demande et un montant déterminés ;
- `PAYMENT_REQUEST` : demande de paiement adressée à un client ;
- `TPE` : encaissement initié depuis un terminal ;
- `MOBILE_MONEY` : débit ou collecte via un opérateur partenaire ;
- `CARD` : paiement traité par un acquéreur ou processeur de cartes ;
- `PUBLIC_SERVICE` : règlement d’une obligation publique.

L’activation de chaque canal dépend du pays, du partenaire, du profil de risque, du KYC, des limites, de la devise et des drapeaux de fonctionnalités.

## 4. Cycle de vie d’un paiement

Les statuts sont :

1. `CREATED` : demande acceptée et identifiant attribué ;
2. `REQUIRES_ACTION` : authentification ou action externe attendue ;
3. `PROCESSING` : traitement ou confirmation partenaire en cours ;
4. `SUCCEEDED` : paiement confirmé et comptabilisé ;
5. `FAILED` : échec définitif ;
6. `CANCELLED` : annulation avant réussite ;
7. `EXPIRED` : délai dépassé ;
8. `REVERSED` : paiement réussi neutralisé par une écriture inverse.

Aucune transition ne peut revenir de `SUCCEEDED` vers `FAILED`. Une correction après succès utilise une annulation ou un remboursement traçable.

## 5. Demandes de paiement

Une demande de paiement contient au minimum :

- le bénéficiaire ou commerçant ;
- le montant et la devise ;
- une description non sensible ;
- une date d’expiration ;
- une référence client facultative ;
- le statut courant.

Le paiement d’une demande crée une opération `Payment` indépendante liée à la demande. Deux tentatives avec la même clé d’idempotence retournent la même opération.

## 6. Transferts

Les types de transfert normalisés sont :

- `INTERNAL` : portefeuille Mansa vers portefeuille Mansa ;
- `BANK` : compte bancaire ;
- `MOBILE_MONEY` : compte opérateur ;
- `MERCHANT` : règlement ou transfert vers commerçant ;
- `PUBLIC_SERVICE` : versement vers un organisme public.

Le parcours recommandé est : devis, création, autorisation forte si nécessaire, exécution, comptabilisation, notification et rapprochement.

Les statuts sont `CREATED`, `PENDING_AUTHORIZATION`, `PENDING`, `PROCESSING`, `COMPLETED`, `FAILED`, `CANCELLED` et `REVERSED`.

## 7. Frais et devis

Un devis fige temporairement :

- le montant principal ;
- les frais de service ;
- les frais partenaire ;
- les taxes ;
- le total débité ;
- le taux de change éventuel ;
- la route partenaire sélectionnée ;
- l’instant d’expiration.

Un devis expiré ne peut pas être utilisé. Toute modification de frais, de partenaire ou de taux nécessite un nouveau devis et une nouvelle validation utilisateur.

## 8. API publique

### Paiements

| Action | Méthode | Route |
|---|---|---|
| Créer un paiement | `POST` | `/v1/payments` |
| Lire un paiement | `GET` | `/v1/payments/:paymentId` |
| Créer une demande de paiement | `POST` | `/v1/payment-requests` |
| Lire une demande | `GET` | `/v1/payment-requests/:paymentRequestId` |
| Payer une demande | `POST` | `/v1/payment-requests/:paymentRequestId/pay` |

### Transferts

| Action | Méthode | Route |
|---|---|---|
| Obtenir un devis | `POST` | `/v1/transfers/quotes` |
| Créer un transfert | `POST` | `/v1/transfers` |
| Lire un transfert | `GET` | `/v1/transfers/:transferId` |
| Autoriser un transfert | `POST` | `/v1/transfers/:transferId/authorize` |
| Annuler un transfert | `POST` | `/v1/transfers/:transferId/cancel` |

Les opérations d’écriture utilisent l’en-tête d’idempotence normalisé, un identifiant de corrélation et le contexte d’authentification. L’accès est limité au propriétaire ou à un acteur explicitement autorisé.

## 9. Routage et tentatives partenaires

Le moteur de routage classe uniquement les routes actives et compatibles avec le pays, la devise, le canal, le montant et les capacités requises. La stratégie peut privilégier le coût, la fiabilité, la latence ou une priorité configurée.

Chaque appel fournisseur crée une `PaymentAttempt` avec son propre statut et sa référence externe. Une nouvelle tentative n’engendre pas une nouvelle opération métier. Les erreurs fonctionnelles définitives ne sont pas relancées automatiquement. Les erreurs techniques temporaires utilisent une politique de reprise plafonnée avec temporisation.

## 10. Contrôles avant exécution

Avant tout débit, le système vérifie :

- l’état du compte, du portefeuille et du bénéficiaire ;
- le niveau KYC ;
- le solde disponible ;
- les limites par opération et période ;
- les règles de risque et conformité ;
- la validité du devis ;
- la devise et le pays ;
- le canal et le partenaire ;
- l’authentification forte requise ;
- l’absence de blocage administratif ou réglementaire.

## 11. Comptabilisation

Une réussite publie atomiquement les écritures nécessaires : montant principal, commission, frais partenaire, taxe, compte de cantonnement et dette envers le bénéficiaire selon le plan comptable validé.

Si le partenaire confirme mais que la publication comptable échoue, l’opération passe en état de traitement contrôlé et déclenche une alerte critique. Elle ne doit pas être déclarée réussie au client tant que la source de vérité financière n’est pas cohérente.

## 12. Événements attendus

- `payment.created` ;
- `payment.action_required` ;
- `payment.processing` ;
- `payment.succeeded` ;
- `payment.failed` ;
- `payment.reversed` ;
- `payment_request.created` ;
- `payment_request.paid` ;
- `payment_request.expired` ;
- `transfer.created` ;
- `transfer.authorization_required` ;
- `transfer.processing` ;
- `transfer.completed` ;
- `transfer.failed` ;
- `transfer.reversed`.

Chaque événement comporte une version de schéma, l’identifiant métier, le pays, la devise, la corrélation et l’horodatage. Aucun secret ou identifiant de paiement complet n’est inclus.

## 13. Critères d’acceptation

- Deux créations identiques avec la même clé d’idempotence produisent un seul paiement ou transfert.
- Une même clé réutilisée avec un contenu différent retourne une erreur de conflit.
- Un devis expiré est refusé.
- Un transfert dépassant une limite ne débite aucun compte.
- Un échec partenaire temporaire peut être relancé sans double débit.
- Un échec fonctionnel définitif n’est pas relancé automatiquement.
- Une réussite partenaire sans comptabilisation déclenche une alerte bloquante.
- Une annulation après succès génère une opération inverse et conserve l’originale.
- Un utilisateur ne peut pas lire ni autoriser l’opération d’un autre utilisateur.
- Les routes, méthodes, types et statuts OpenAPI correspondent aux contrats TypeScript.
- Les applications Client, Commerçant et TPE consomment les mêmes identifiants et statuts.
- Les journaux ne contiennent ni PAN complet, ni PIN, ni OTP, ni secret partenaire.

## 14. Éléments à configurer avant production

Les barèmes de frais, taxes, plafonds, règles de change, partenaires, clés d’intégration, certificats, comptes de cantonnement, délais d’expiration, politiques de reprise, seuils d’authentification forte et procédures de rapprochement doivent être validés par pays avec les banques, opérateurs, acquéreurs, autorités et équipes conformité.
