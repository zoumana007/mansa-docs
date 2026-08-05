# Objectifs d’épargne et coffres

## Objectif

Le module d’épargne permet à un client de créer des objectifs financiers, d’y affecter des contributions manuelles ou automatiques et de suivre leur progression sans mélanger les montants réservés avec le solde librement disponible. Le grand livre reste la source de vérité : un objectif ne modifie jamais un solde par simple mise à jour d’un compteur.

## Concepts

Un objectif d’épargne contient :

- un propriétaire et un portefeuille de rattachement ;
- un nom affiché au client ;
- un montant cible ;
- le montant effectivement épargné ;
- une date cible facultative ;
- un statut et des horodatages d’audit.

Les montants utilisent le type monétaire partagé et ne sont jamais représentés en nombre flottant.

## Statuts

- `ACTIVE` : objectif ouvert aux contributions ;
- `PAUSED` : contributions automatiques suspendues, consultation maintenue ;
- `COMPLETED` : montant cible atteint et objectif clôturé avec succès ;
- `CANCELLED` : objectif fermé par le client ou par une décision autorisée.

`COMPLETED` et `CANCELLED` sont définitifs. Une annulation ne détruit pas l’historique et ne doit pas supprimer les écritures comptables associées.

## Sources de contribution

- `MANUAL` : versement demandé explicitement par le client ;
- `SCHEDULED` : versement déclenché par une règle planifiée ;
- `ROUND_UP` : arrondi d’un paiement selon la configuration du client ;
- `CASHBACK` : affectation d’un avantage ou remboursement commercial.

Chaque contribution référence une transaction comptable et une clé d’idempotence. Une répétition de la même commande ne doit jamais produire deux débits.

## Règles métier

1. La devise de la contribution doit correspondre à celle de l’objectif et du portefeuille source.
2. Le montant doit être strictement positif.
3. Un objectif `PAUSED`, `COMPLETED` ou `CANCELLED` refuse une nouvelle contribution, sauf opération corrective interne dûment auditée.
4. Le solde disponible doit être suffisant au moment de l’autorisation et de l’écriture finale.
5. L’atteinte ou le dépassement du montant cible fait passer l’objectif à `COMPLETED` selon une transaction atomique.
6. Une modification du montant cible ne peut pas rendre incohérent un objectif déjà complété.
7. Toute automatisation doit pouvoir être désactivée immédiatement depuis l’application ou l’administration autorisée.
8. Les règles de frais, limites, KYC et risque s’appliquent comme pour tout mouvement financier.

## Parcours client

### Création

Le client choisit un nom, un portefeuille, un montant cible et éventuellement une date cible. L’application présente un résumé avant confirmation et utilise une clé d’idempotence lors de l’envoi.

### Contribution

Le client sélectionne le montant et le portefeuille source. L’interface affiche l’effet sur le solde disponible et la progression attendue. Le résultat final n’est confirmé qu’après réponse du backend.

### Pause et reprise

La pause suspend les mécanismes automatiques mais ne libère ni ne supprime les fonds déjà affectés. La reprise réactive uniquement les règles encore valides.

### Annulation

L’application explique le traitement des fonds avant confirmation. La libération, le transfert ou le maintien des fonds dépend du modèle comptable adopté et doit être matérialisé par des écritures explicites.

## API alignée avec le dépôt plateforme

Contrats maintenus dans :

- `packages/contracts/src/savings-goal.ts` ;
- `packages/contracts/src/savings-goal-api.ts`.

Routes principales :

- `GET /v1/savings-goals` ;
- `POST /v1/savings-goals` ;
- `GET /v1/savings-goals/:goalId` ;
- `PATCH /v1/savings-goals/:goalId` ;
- `POST /v1/savings-goals/:goalId/contributions` ;
- `GET /v1/savings-goals/:goalId/contributions`.

La création d’un objectif et chaque contribution sont idempotentes. Les listes sont paginées et filtrables par propriétaire, portefeuille et statut.

## Sécurité et conformité

- Authentification renforcée pour les opérations sensibles ou montants élevés.
- Vérification des permissions sur le propriétaire et le portefeuille à chaque appel.
- Journal d’audit pour création, modification, pause, reprise, annulation et contribution.
- Aucune donnée bancaire sensible dans les noms, descriptions, notifications ou journaux.
- Limitation de fréquence et contrôles antifraude sur les contributions répétées.
- Corrélation obligatoire entre contribution, transaction, écriture de grand livre et notification.

## Critères de recette

1. Une création répétée avec la même clé d’idempotence ne crée qu’un objectif.
2. Une contribution répétée avec la même clé ne débite qu’une fois le portefeuille.
3. Une contribution dans une devise différente est rejetée.
4. Un objectif en pause refuse les contributions automatiques.
5. Un objectif complété ou annulé ne peut pas revenir à `ACTIVE`.
6. Le montant épargné correspond exactement aux transactions comptabilisées.
7. L’atteinte du montant cible clôture l’objectif de manière atomique.
8. Les listes sont paginées et ne révèlent que les objectifs accessibles à l’acteur.
9. Toute mutation produit un événement d’audit corrélable.
10. Aucun secret ni identifiant de paiement sensible n’apparaît dans les réponses ou journaux.
