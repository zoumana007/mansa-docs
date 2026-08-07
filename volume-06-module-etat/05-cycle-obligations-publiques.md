# Cycle des obligations publiques

## 1. Objet

Ce document définit le cycle de vie des obligations publiques Mansa : amendes, taxes, frais, scolarité, immatriculation, licences et autres créances administratives. Il complète les contrats `public-services.ts` et `public-services-api.ts` de `mansa-platform`.

L’objectif est de rendre chaque création, émission, contestation, annulation et encaissement traçable, idempotent et soumis aux habilitations de l’organisme concerné.

## 2. États

Une obligation utilise les états canoniques suivants :

- `DRAFT` : préparée mais non opposable au citoyen ;
- `ISSUED` : officiellement émise ;
- `PARTIALLY_PAID` : paiement partiel reçu lorsque le service le permet ;
- `PAID` : montant intégral encaissé ;
- `OVERDUE` : échéance dépassée ;
- `DISPUTED` : contestation ouverte ;
- `CANCELLED` : obligation annulée par une autorité habilitée ;
- `EXPIRED` : obligation arrivée à expiration selon la règle applicable ;
- `REFUNDED` : montant remboursé selon le processus financier autorisé.

Aucun changement d’état ne doit supprimer l’historique précédent.

## 3. Création et émission

`CreatePublicObligationCommand` crée l’obligation de manière idempotente à partir d’une entrée versionnée du catalogue public.

L’émission est une action distincte via `IssuePublicObligationCommand` et la route :

`POST /v1/public-services/obligations/:obligationId/issuance`

L’émission exige :

1. un agent public actif ;
2. une juridiction compatible ;
3. une permission d’émission ;
4. un appareil autorisé lorsque cette politique est activée ;
5. une clé d’idempotence ;
6. un événement d’audit corrélé.

Pour une amende terrain, le montant ne doit jamais être choisi librement par l’agent lorsque le catalogue fixe un barème. Le terminal applique la version active du service.

## 4. Contestation

Une contestation utilise `DisputePublicObligationCommand` et :

`POST /v1/public-services/obligations/:obligationId/disputes`

Le motif est obligatoire. Le demandeur peut être un utilisateur ou un agent habilité. Des références vers des pièces justificatives peuvent être associées sans placer les fichiers eux-mêmes dans le contrat de transport.

La contestation :

- place l’obligation dans un état permettant son examen ;
- ne modifie pas rétroactivement son montant ou son motif initial ;
- conserve l’identité de l’acteur, la date et la corrélation ;
- applique la politique de paiement pendant contestation définie par l’organisme.

## 5. Annulation

Une annulation utilise `CancelPublicObligationCommand` et :

`POST /v1/public-services/obligations/:obligationId/cancellation`

L’annulation est une opération à risque élevé. Elle exige obligatoirement :

- un agent demandeur identifié ;
- une justification ;
- un `approvalRequestId` ;
- une validation conforme à la séparation des responsabilités.

L’agent qui a émis une amende ne doit pas pouvoir l’annuler seul lorsque la politique de l’organisme exige une double validation.

Une obligation déjà encaissée ne doit pas être simplement annulée pour restituer des fonds : elle doit suivre le processus de remboursement afin de préserver la cohérence du grand livre.

## 6. Encaissement

Le paiement reste exposé via :

`POST /v1/public-services/obligations/:obligationId/payments`

`CollectPublicPaymentCommand` contient une clé d’idempotence, le montant, le canal de paiement et, lorsque pertinent, le payeur, l’agent collecteur et le terminal.

Chaque encaissement réussi génère un `PublicPaymentReceipt` contenant notamment la référence de l’obligation, l’identifiant de transaction et un code de vérification.

Le reçu doit pouvoir être vérifié indépendamment par le citoyen sans exposer de donnée personnelle excessive.

## 7. Contrôles anti-corruption et anti-fraude

- Les barèmes sont versionnés et administrés centralement.
- Un agent terrain ne peut pas modifier localement un montant officiel.
- Une annulation à risque élevé requiert une approbation séparée.
- Les opérations sensibles sont rattachées à l’agent, au terminal et à la juridiction.
- Les commandes rejouables utilisent une clé d’idempotence.
- Les événements d’audit sont conservés même après suspension ou révocation d’un agent.
- Une correction administrative ne doit jamais supprimer silencieusement une opération financière.

## 8. Cohérence financière

Le montant payé ne peut pas dépasser le solde restant sauf mécanisme explicite de trop-perçu.

Les paiements partiels ne sont acceptés que si l’entrée de catalogue définit `partialPaymentAllowed = true`.

Toute restitution de fonds doit créer une opération financière de remboursement liée à la transaction d’origine et non réécrire l’historique.

## 9. Critères de recette

1. Les routes d’émission, contestation et annulation correspondent à `PUBLIC_SERVICES_API_ROUTES`.
2. `IssuePublicObligationCommand`, `DisputePublicObligationCommand` et `CancelPublicObligationCommand` sont disponibles dans les contrats partagés.
3. Une annulation exige un `approvalRequestId`.
4. Les trois commandes sont idempotentes.
5. Une obligation payée ne peut pas être effacée ni transformée en simple annulation pour rembourser le citoyen.
6. L’historique d’état et l’audit restent disponibles après toute transition.
7. Les règles d’habilitation des agents décrites dans `04-agents-publics-et-habilitations.md` s’appliquent aux opérations terrain.
8. La documentation et les contrats TypeScript ne présentent aucun écart connu sur ces transitions.
