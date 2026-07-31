# Format standard des erreurs API

## Objectif

Toutes les erreurs HTTP exposées par l’API Gateway Mansa utilisent une enveloppe stable, exploitable par les applications mobiles, les portails web, le support et l’observabilité.

## Contrat de réponse

```json
{
  "error": {
    "code": "BAD_REQUEST",
    "message": "La requête contient des données invalides.",
    "details": ["phoneNumber must be a valid phone number"]
  },
  "meta": {
    "correlationId": "9e5f3f36-49d5-4d07-b809-78789ba19ab7",
    "timestamp": "2026-07-31T07:40:00.000Z",
    "path": "/api/v1/accounts"
  }
}
```

`details` est facultatif. Il est principalement utilisé pour les erreurs de validation. Les clients ne doivent jamais déduire une règle métier du texte de `message` : seul `error.code` constitue un identifiant stable.

## Règles

- `error.code` est une chaîne machine stable en majuscules avec séparateurs `_`.
- `error.message` est un message compréhensible, traduisible et sans information sensible.
- `error.details` peut contenir des précisions structurées non sensibles.
- `meta.correlationId` reprend l’identifiant de corrélation de la requête.
- `meta.timestamp` est exprimé en UTC au format ISO 8601.
- `meta.path` contient le chemin HTTP reçu, sans exposer de secret.
- Une erreur inconnue retourne le statut `500`, le code `INTERNAL_SERVER_ERROR` et un message générique.
- Les traces techniques, piles d’appels, requêtes SQL, jetons et secrets ne sont jamais renvoyés au client.

## Implémentation initiale

Le filtre NestJS global est défini dans :

```text
apps/api-gateway/src/http-exception.filter.ts
```

Il est enregistré avec `APP_FILTER` dans :

```text
apps/api-gateway/src/app.module.ts
```

Le filtre normalise les `HttpException` NestJS et masque les exceptions inconnues. Les erreurs de validation dont le champ `message` est une liste produisent un message général et placent la liste dans `error.details`.

## Catalogue initial

| Statut | Code par défaut | Usage |
|---:|---|---|
| 400 | `BAD_REQUEST` | Données invalides ou commande incohérente |
| 401 | `UNAUTHORIZED` | Authentification absente ou invalide |
| 403 | `FORBIDDEN` | Permission insuffisante |
| 404 | `NOT_FOUND` | Ressource absente |
| 409 | `CONFLICT` | Conflit métier ou ressource déjà existante |
| 422 | `UNPROCESSABLE_ENTITY` | Règle métier non satisfaite |
| 429 | `TOO_MANY_REQUESTS` | Limite de débit dépassée |
| 500 | `INTERNAL_SERVER_ERROR` | Erreur interne masquée |
| 503 | `SERVICE_UNAVAILABLE` | Dépendance ou service temporairement indisponible |

Les modules métier pourront fournir un code plus précis dans la réponse de leur `HttpException`, par exemple `ACCOUNT_BLOCKED`, `INSUFFICIENT_FUNDS` ou `KYC_REQUIRED`.

## Critères d’acceptation

- Toutes les routes utilisent la même enveloppe d’erreur.
- Le statut HTTP reste cohérent avec la catégorie d’erreur.
- Une exception inconnue ne révèle aucun détail technique.
- L’identifiant de corrélation est présent quand l’intercepteur l’a attaché à la requête.
- Une erreur de validation conserve la liste des champs invalides dans `details`.
- Les applications clientes peuvent afficher `message` et piloter leur logique avec `code`.

## Prochaines étapes

1. Ajouter des tests unitaires de normalisation et des tests d’intégration HTTP.
2. Définir un registre partagé des codes d’erreur métier.
3. Ajouter la localisation des messages côté client ou passerelle.
4. Journaliser les exceptions avec le même identifiant de corrélation.
5. Documenter les erreurs de chaque opération dans le futur contrat OpenAPI.
