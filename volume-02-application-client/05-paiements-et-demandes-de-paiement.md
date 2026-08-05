# Paiements et demandes de paiement

## 1. Objectif

Ce document définit le contrat fonctionnel commun aux paiements initiés par l’application Client et aux demandes de paiement reçues ou émises. Il s’applique également aux interfaces Commerçant, TPE, Admin et aux intégrations partenaires lorsqu’elles consomment les mêmes ressources.

Le contrat technique correspondant est maintenu dans `mansa-platform/packages/contracts/src/payment-api.ts`.

## 2. Canaux supportés

Les canaux normalisés sont :

- `INTERNAL` : paiement entre portefeuilles Mansa ;
- `QR_STATIC` : paiement d’un bénéficiaire à partir d’un QR permanent ;
- `QR_DYNAMIC` : paiement d’une intention contenant montant, devise et expiration ;
- `PAYMENT_REQUEST` : règlement d’une demande de paiement ;
- `TPE` : opération initiée depuis un terminal ;
- `MOBILE_MONEY` : opération routée vers un opérateur Mobile Money ;
- `CARD` : opération impliquant un réseau ou processeur carte ;
- `PUBLIC_SERVICE` : paiement d’une obligation ou d’un service public.

Le canal ne détermine pas à lui seul le partenaire final. Le routage reste une décision du backend selon le pays, la devise, le bénéficiaire, la disponibilité, les coûts et les règles de risque.

## 3. Création d’un paiement

La création utilise `POST /v1/payments` avec une clé d’idempotence obligatoire.

Les données minimales sont :

- portefeuille payeur ;
- bénéficiaire ;
- montant en unités mineures et devise ;
- canal ;
- clé d’idempotence.

Le client peut ajouter une description, une référence métier et un identifiant d’intention. Une nouvelle tentative avec la même clé et la même charge utile doit retourner le résultat initial. Une réutilisation de la même clé avec des données différentes doit être rejetée.

Avant acceptation, le backend vérifie au minimum :

1. l’existence et le statut des acteurs et portefeuilles ;
2. les droits de l’acteur ;
3. la devise et les capacités du canal ;
4. les limites configurées ;
5. le solde disponible ou la source externe autorisée ;
6. les règles de conformité, fraude et sanctions ;
7. les frais applicables ;
8. l’absence de doublon fonctionnel évident.

## 4. Cycle de vie

Les statuts possibles sont :

- `CREATED` ;
- `REQUIRES_ACTION` ;
- `PROCESSING` ;
- `SUCCEEDED` ;
- `FAILED` ;
- `CANCELLED` ;
- `EXPIRED` ;
- `REVERSED`.

Un paiement réussi ne peut pas être simplement supprimé ou remis à l’état initial. Toute correction financière ultérieure doit utiliser une opération de reversement ou de remboursement traçable.

Les transitions sont décidées par le backend et journalisées. Les applications ne doivent jamais déduire un succès définitif uniquement à partir d’un écran local ou d’une réponse réseau interrompue.

## 5. Consultation et filtrage

Les routes principales sont :

- `GET /v1/payments` ;
- `GET /v1/payments/:paymentId` ;
- `GET /v1/payments/:paymentId/receipt`.

La liste est paginée et filtrable par portefeuille payeur, bénéficiaire, portefeuille bénéficiaire, statut, canal, référence client et période de création.

Les autorisations doivent empêcher un utilisateur de consulter une opération étrangère à son périmètre. Les administrateurs utilisent des permissions et portées explicites ; toute consultation sensible peut être auditée.

## 6. Annulation

`POST /v1/payments/:paymentId/cancel` demande l’annulation d’un paiement encore annulable.

La requête contient un motif obligatoire. Le backend refuse notamment l’annulation lorsque :

- le paiement est déjà final ;
- le partenaire externe a rendu l’ordre irréversible ;
- une écriture financière définitive impose désormais un reversement ;
- l’acteur n’a pas le droit d’annuler l’opération.

Une annulation concurrente doit rester idempotente au niveau métier.

## 7. Reversement

`POST /v1/payments/:paymentId/reverse` crée la correction d’un paiement éligible. La requête exige un motif et une clé d’idempotence.

Le reversement :

- ne modifie pas silencieusement les écritures d’origine ;
- produit ses propres écritures en partie double ;
- conserve le lien vers le paiement initial ;
- recalcule ou reprend les frais selon la politique applicable ;
- est soumis aux permissions, limites et éventuelle double validation ;
- reste traçable dans l’historique utilisateur et les journaux administratifs.

Le remboursement partiel et les litiges sont gérés par leurs modules spécialisés et ne doivent pas être simulés par une modification manuelle du montant du paiement d’origine.

## 8. Reçu

Le reçu retourné par `GET /v1/payments/:paymentId/receipt` contient uniquement les informations autorisées : référence, date, montant, devise, frais, canal, statut, libellé, contrepartie masquée et éléments de vérification nécessaires.

Il ne doit jamais exposer :

- numéro complet de carte ;
- code de sécurité ;
- secret partenaire ;
- document KYC ;
- donnée personnelle non nécessaire.

Le reçu affiché ou exporté ne remplace pas le grand livre ni les preuves conservées côté serveur.

## 9. Demandes de paiement

Les routes sont :

- `GET /v1/payment-requests` ;
- `POST /v1/payment-requests` ;
- `GET /v1/payment-requests/:paymentRequestId` ;
- `POST /v1/payment-requests/:paymentRequestId/pay` ;
- `POST /v1/payment-requests/:paymentRequestId/cancel`.

Une demande de paiement précise le demandeur, le montant, la devise, l’éventuel payeur visé, le libellé et l’expiration. Elle ne réserve pas automatiquement les fonds du payeur.

La liste est paginée et filtrable par demandeur, payeur, statut, période de création et date d’expiration.

Le règlement d’une demande :

1. vérifie qu’elle est toujours payable ;
2. vérifie le payeur et son portefeuille ;
3. utilise une clé d’idempotence ;
4. crée un paiement normal avec le canal `PAYMENT_REQUEST` ;
5. lie le paiement à la demande ;
6. empêche un double règlement sauf comportement explicitement configuré pour une demande fractionnable future.

L’annulation d’une demande exige un motif et n’annule pas rétroactivement un paiement déjà réussi.

## 10. Sécurité et audit

Chaque création, annulation, reversement et règlement doit enregistrer :

- acteur et type d’acteur ;
- ressource ciblée ;
- résultat ;
- identifiant de corrélation ;
- clé d’idempotence lorsqu’elle existe ;
- horodatage serveur ;
- environnement ;
- motif ou code d’échec non sensible.

Les journaux ne doivent contenir ni secret, ni OTP, ni numéro de carte complet, ni charge utile partenaire brute contenant des données sensibles.

## 11. Critères d’acceptation

- Deux requêtes identiques avec la même clé ne débitent jamais deux fois.
- Une clé réutilisée avec une charge utile différente est rejetée.
- Les listes sont paginées et limitées au périmètre autorisé.
- Un paiement final ne peut pas être annulé comme une opération en attente.
- Un reversement produit une opération liée et des écritures distinctes.
- Une demande expirée ou annulée ne peut pas être payée.
- Deux règlements concurrents de la même demande non fractionnable ne produisent qu’un seul paiement réussi.
- Le reçu masque les données sensibles.
- Les erreurs partenaires restent corrélables sans exposer de secret.
- Toutes les actions critiques sont auditables.

## 12. Éléments restant à implémenter

Ce contrat décrit l’interface cible. Restent à construire progressivement dans le dépôt plateforme : contrôleurs NestJS, services d’application, persistance, machine d’état, écritures ledger, politiques d’autorisation, intégrations de routage, événements, tests contractuels et parcours d’interface.
