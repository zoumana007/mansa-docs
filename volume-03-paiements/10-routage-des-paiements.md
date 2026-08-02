# Routage des paiements

## Objet

Le routage des paiements choisit le prestataire technique qui doit traiter une opération lorsqu’un même produit peut passer par plusieurs banques, processeurs, opérateurs Mobile Money ou réseaux de cartes.

## Principes

- La sélection est déterministe et explicable.
- Une route désactivée ou dégradée n’est jamais choisie comme route normale.
- Le pays, la devise, le type d’opération, le canal et les limites de montant sont vérifiés avant classement.
- Une route ne contient aucun secret d’intégration ; elle référence uniquement un prestataire configuré dans l’environnement.
- La route choisie et la liste ordonnée des routes admissibles sont conservées avec la tentative de paiement.
- Un nouvel essai ne doit pas changer silencieusement de prestataire lorsqu’une autorisation financière a déjà été obtenue.

## États

- `ACTIVE` : route autorisée pour les nouvelles opérations.
- `DEGRADED` : route connue comme instable et exclue de la sélection normale.
- `DISABLED` : route interdite.

Le passage vers `DEGRADED` ou `DISABLED` doit être journalisé. Une désactivation d’urgence peut être immédiate, mais sa justification et son auteur restent obligatoires.

## Stratégies

### Priorité

`PRIORITY` sélectionne la valeur numérique la plus faible. Cette stratégie est adaptée aux accords contractuels imposant un prestataire principal et des solutions de secours.

### Coût le plus faible

`LOWEST_COST` classe les routes selon leur coût estimé en unités mineures. La priorité départage les coûts identiques.

Le coût estimé sert au routage ; la tarification client reste régie par la politique de frais et doit être calculée séparément.

### Disponibilité la plus élevée

`HIGHEST_AVAILABILITY` sélectionne la meilleure disponibilité exprimée en points de base. La mesure utilisée doit provenir d’une fenêtre documentée et ne doit pas dépendre d’un seul succès récent.

## Filtres d’admissibilité

Une route est admissible lorsque :

1. son état est `ACTIVE` ;
2. son type d’opération correspond ;
3. son pays et sa devise correspondent ;
4. son canal correspond lorsqu’il est imposé ;
5. le montant respecte les limites minimales et maximales.

En l’absence de route admissible, l’opération est refusée avec une erreur métier contrôlée. Il est interdit de choisir arbitrairement une route incompatible.

## Basculement et reprises

Le service applicatif peut tenter la route suivante seulement lorsque :

- l’échec est classé comme temporaire ou technique ;
- aucune autorisation irréversible n’a été confirmée ;
- la clé d’idempotence et l’identifiant de tentative sont conservés ;
- la politique de reprise limite le nombre d’essais ;
- chaque tentative est auditée avec le motif du basculement.

Les refus métier, fonds insuffisants, limites réglementaires, fraude ou authentification échouée ne doivent pas déclencher automatiquement un changement de prestataire.

## Administration

La configuration des routes doit être versionnée et soumise à des permissions dédiées. Les changements de priorité, limites, coût estimé et état nécessitent un audit. Une modification de production à fort impact doit utiliser une double validation.

Les secrets, URL privées et certificats restent dans le gestionnaire de secrets ou la configuration d’environnement. La route conserve seulement un `providerId` logique.

## Observabilité

Les métriques minimales sont :

- taux d’acceptation et d’erreur par route ;
- latence par prestataire et type d’opération ;
- nombre de basculements ;
- coût estimé et coût réel ;
- disponibilité calculée ;
- opérations sans route admissible.

Les tableaux de bord doivent distinguer un problème partenaire d’un refus métier utilisateur.

## Alignement avec le code

Le contrat de référence est `packages/contracts/src/payment-routing.ts` dans `mansa-platform`.

Il expose :

- `PAYMENT_ROUTE_STATUSES` et `PAYMENT_ROUTE_STRATEGIES` ;
- `PaymentRouteCandidate`, `SelectPaymentRouteCommand` et `PaymentRouteSelection` ;
- `selectPaymentRoute` et les garde-types ;
- le sous-chemin public `@mansa/contracts/payment-routing`.

## Critères d’acceptation

- Les routes inactives, incompatibles ou hors limites sont exclues.
- Chaque stratégie produit un classement déterministe.
- La priorité départage les égalités.
- Un montant négatif est refusé.
- L’absence de route admissible produit une erreur contrôlée.
- Les tests couvrent priorité, coût, disponibilité et exclusions.
- La documentation et les noms exportés correspondent au code.
