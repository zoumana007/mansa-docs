# Conventions API, erreurs et corrélation

## Objectif

Ce document fixe les règles communes à toutes les API Mansa afin que les applications mobiles, les interfaces web, les terminaux, les workers et les partenaires utilisent des contrats prévisibles et auditables.

## Versionnement

- Les API publiques sont préfixées par `/v1`.
- Une rupture de contrat nécessite une nouvelle version majeure.
- L’ajout d’un champ optionnel est compatible ; le retrait ou le changement de sens d’un champ ne l’est pas.
- Les contrats partagés sont définis dans `packages/contracts` et exportés explicitement.

## Identifiants de requête

Chaque requête possède :

- `requestId` : identifiant unique créé à l’entrée de la plateforme ;
- `correlationId` : identifiant commun à toute une chaîne d’opérations ;
- `causationId` : identifiant de l’événement ou de la commande à l’origine d’un traitement asynchrone ;
- `traceId` : identifiant de trace distribué lorsque l’observabilité est activée.

Le client peut fournir `x-correlation-id`. La passerelle le valide ou en génère un nouveau. Les services internes propagent ces valeurs sans les remplacer.

## Idempotence

Toute opération susceptible de créer un mouvement financier, une obligation, un paiement, une carte, un remboursement ou une instruction partenaire exige un en-tête `Idempotency-Key`.

Règles :

1. La clé est liée à l’acteur, à l’opération et à l’environnement.
2. Une répétition avec la même charge retourne le résultat initial.
3. Une répétition avec une charge différente retourne une erreur de conflit.
4. La durée de conservation est configurable, avec une valeur cible minimale de 24 heures pour les opérations non financières et supérieure pour les opérations financières.
5. La réponse réutilisée conserve la référence métier initiale.

## Format de succès

Les ressources simples sont retournées directement avec les métadonnées de requête dans les en-têtes. Les listes utilisent :

```json
{
  "items": [],
  "pageInfo": {
    "nextCursor": null,
    "hasNextPage": false
  }
}
```

La pagination par curseur est préférée pour les transactions et journaux. La pagination par page reste réservée aux catalogues stables et petits jeux de données.

## Format d’erreur

```json
{
  "code": "VALIDATION_ERROR",
  "message": "La requête contient des données invalides.",
  "requestId": "req_...",
  "correlationId": "cor_...",
  "details": [],
  "retryable": false
}
```

Le message ne doit jamais révéler de secret, pile d’exécution, requête SQL, jeton, donnée KYC ou détail d’un système partenaire.

## Codes HTTP

- `200` lecture ou action terminée ;
- `201` ressource créée ;
- `202` traitement accepté et asynchrone ;
- `204` action terminée sans contenu ;
- `400` requête invalide ;
- `401` authentification absente ou invalide ;
- `403` droit insuffisant ;
- `404` ressource inaccessible ou inexistante ;
- `409` conflit, doublon ou idempotence incohérente ;
- `422` règle métier non satisfaite ;
- `429` limite dépassée ;
- `500` erreur interne non détaillée ;
- `502` ou `503` dépendance indisponible ;
- `504` délai partenaire dépassé.

## Temps, montants et devises

- Les dates sont en UTC au format ISO 8601.
- Les montants sont des entiers en unités mineures.
- La devise est un code ISO 4217 explicite.
- Un montant et sa devise ne sont jamais séparés dans un contrat métier.
- Les fuseaux locaux servent uniquement à l’affichage et aux règles calendaires configurées.

## Webhooks

Chaque webhook sortant contient un identifiant d’événement, un type versionné, une date, une charge, un environnement et les identifiants de corrélation. Il est signé, rejouable, journalisé et livré avec reprise exponentielle. Le destinataire doit traiter les doublons par identifiant d’événement.

## Journalisation

Les logs structurés comprennent au minimum le niveau, le service, l’environnement, `requestId`, `correlationId`, l’acteur pseudonymisé, l’opération et le résultat. Les numéros de carte, secrets, mots de passe, OTP, documents KYC et données sensibles ne sont jamais journalisés.

## Critères d’acceptation

- Tous les points d’entrée génèrent ou propagent les identifiants de corrélation.
- Les erreurs respectent un contrat partagé et ne divulguent aucune donnée sensible.
- Les commandes financières refusent l’absence de clé d’idempotence.
- Les applications peuvent afficher un identifiant de support à partir de `requestId`.
- Les workers propagent `correlationId` et `causationId` dans leurs événements.
- Les contrats TypeScript correspondants sont exportés par `@mansa/contracts`.
