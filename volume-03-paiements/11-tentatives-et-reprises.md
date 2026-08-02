# Tentatives de paiement et reprises

## Objet

Une tentative de paiement représente un appel unique vers une route et un prestataire déterminés. Elle permet de distinguer l’intention de paiement métier des appels techniques successifs nécessaires à son traitement.

Un paiement peut posséder plusieurs tentatives, mais une tentative ne peut concerner qu’un seul prestataire et une seule route.

## Principes

- Chaque tentative possède un identifiant unique et une clé d’idempotence.
- Le numéro de séquence commence à `1` et augmente pour chaque nouvelle tentative du même paiement.
- Le prestataire et la route sont figés après la création.
- Une tentative terminée ne peut pas être réouverte.
- Les détails d’échec sont conservés sans exposer de secret ni de donnée sensible.
- Un nouvel essai n’est autorisé que si la catégorie d’échec le permet et si aucune confirmation financière irréversible n’a été obtenue.
- L’état `UNKNOWN` déclenche une réconciliation avant toute nouvelle tentative susceptible de produire un double débit.

## États

- `CREATED` : tentative enregistrée mais non envoyée.
- `PROCESSING` : appel prestataire en cours ou accepté techniquement.
- `AUTHORIZED` : autorisation financière obtenue, capture ou confirmation finale encore attendue.
- `SUCCEEDED` : paiement confirmé par le prestataire.
- `FAILED` : échec final classé.
- `CANCELLED` : tentative annulée avant confirmation financière.
- `UNKNOWN` : résultat incertain nécessitant interrogation ou réconciliation.

Les états finaux sont `SUCCEEDED`, `FAILED` et `CANCELLED`.

## Transitions autorisées

| État courant | États suivants autorisés |
|---|---|
| `CREATED` | `PROCESSING`, `CANCELLED` |
| `PROCESSING` | `AUTHORIZED`, `SUCCEEDED`, `FAILED`, `UNKNOWN`, `CANCELLED` |
| `AUTHORIZED` | `SUCCEEDED`, `FAILED`, `UNKNOWN` |
| `UNKNOWN` | `AUTHORIZED`, `SUCCEEDED`, `FAILED`, `CANCELLED` |
| `SUCCEEDED` | aucun |
| `FAILED` | aucun |
| `CANCELLED` | aucun |

Toute transition non prévue est rejetée et journalisée comme anomalie applicative.

## Catégories d’échec

- `BUSINESS` : refus métier du prestataire, fonds insuffisants ou instrument non accepté.
- `AUTHENTICATION` : authentification ou validation forte échouée.
- `LIMIT` : plafond réglementaire, contractuel ou utilisateur dépassé.
- `FRAUD` : blocage risque ou fraude.
- `TEMPORARY` : indisponibilité temporaire explicitement signalée.
- `TECHNICAL` : erreur réseau, protocole ou infrastructure.
- `UNKNOWN` : cause non déterminée ; reprise interdite sans vérification complémentaire.

La catégorie est obligatoire lorsque la tentative passe à `FAILED`. Un code prestataire normalisé peut compléter la catégorie.

## Politique de reprise

Une nouvelle tentative peut être envisagée pour `TEMPORARY`, `TECHNICAL` ou `UNKNOWN`, sous réserve des contrôles suivants :

1. le paiement n’est pas déjà confirmé ;
2. aucune autorisation irréversible n’est active ;
3. la tentative précédente a été interrogée ou réconciliée lorsque son état est incertain ;
4. le nombre maximal d’essais n’est pas dépassé ;
5. la même clé d’idempotence métier reste associée au paiement ;
6. une nouvelle clé technique ou un suffixe de séquence conforme au contrat du prestataire est utilisé si nécessaire ;
7. le changement de route est permis par la politique de routage.

Les catégories `BUSINESS`, `AUTHENTICATION`, `LIMIT` et `FRAUD` ne déclenchent pas de reprise automatique.

## Idempotence

La plateforme conserve deux niveaux :

- une clé d’idempotence métier pour l’intention de paiement ;
- un identifiant unique par tentative technique.

Une répétition du même appel ne doit pas créer une nouvelle tentative si le couple paiement–séquence existe déjà. Le prestataire doit recevoir une clé stable compatible avec son protocole.

## Réconciliation

Une tentative `UNKNOWN` est placée dans une file de réconciliation. Le service interroge le prestataire à partir de la référence externe lorsqu’elle existe. Les réponses tardives et webhooks sont corrélés avec l’identifiant de tentative, le paiement et la route.

Aucun nouveau débit ne doit être lancé tant que le risque de double traitement n’est pas écarté.

## Audit et observabilité

Chaque création et transition conserve :

- l’identifiant du paiement et de la tentative ;
- le numéro de séquence ;
- la route et le prestataire ;
- l’ancien et le nouvel état ;
- la catégorie et le code d’échec ;
- la référence prestataire ;
- les horodatages ;
- l’identifiant de corrélation.

Les métriques minimales couvrent les tentatives par paiement, transitions vers `UNKNOWN`, taux de reprise, succès après reprise, doubles réponses et latence par prestataire.

## Alignement avec le code

Le contrat de référence est `packages/contracts/src/payment-attempt.ts` dans `mansa-platform`.

Il expose :

- `PAYMENT_ATTEMPT_STATUSES` et `PAYMENT_FAILURE_CATEGORIES` ;
- `PaymentAttempt`, `CreatePaymentAttemptCommand` et `TransitionPaymentAttemptCommand` ;
- `createPaymentAttempt` et `transitionPaymentAttempt` ;
- `canTransitionPaymentAttempt`, `isFinalPaymentAttemptStatus` et `isRetryablePaymentFailure` ;
- le sous-chemin public `@mansa/contracts/payment-attempt`.

## Critères d’acceptation

- Une tentative est créée dans l’état `CREATED` avec une séquence positive.
- Les transitions non autorisées sont rejetées.
- Les états finaux ne peuvent plus évoluer.
- Une catégorie d’échec est obligatoire pour `FAILED`.
- Les détails d’échec sont interdits sur les états non échoués.
- Les reprises automatiques sont limitées aux catégories prévues.
- Les tests couvrent création, transitions, états finaux, échecs et séquences invalides.
- Les noms documentés correspondent aux exports du paquet de contrats.
