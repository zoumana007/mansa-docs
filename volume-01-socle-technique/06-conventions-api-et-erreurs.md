# Volume 1 — Conventions API et gestion des erreurs

## 1. Versionnement

Toutes les routes publiques sont exposées sous `/api/v1`. Une rupture de contrat exige une nouvelle version majeure. Les ajouts rétrocompatibles restent dans la version active.

## 2. Format des requêtes

- JSON UTF-8 pour les API métier.
- Horodatages au format ISO 8601 en UTC.
- Montants en unités mineures sous forme d’entiers accompagnés d’un code devise ISO 4217.
- Identifiants techniques opaques et non séquentiels.
- En-tête `Idempotency-Key` obligatoire pour toute création ou opération financière susceptible d’être rejouée.
- En-tête `X-Correlation-Id` accepté depuis les systèmes partenaires et généré lorsqu’il est absent.

## 3. Réponses

Une réponse réussie contient directement la ressource ou un objet paginé. Les listes utilisent un curseur plutôt qu’un numéro de page lorsque l’ordre peut évoluer rapidement.

Exemple :

```json
{
  "items": [],
  "nextCursor": null
}
```

## 4. Erreurs

Les erreurs utilisent un format stable :

```json
{
  "code": "INSUFFICIENT_FUNDS",
  "message": "Solde insuffisant",
  "correlationId": "opaque-id",
  "details": []
}
```

`code` est destiné aux applications et reste stable. `message` peut être localisé. `details` ne doit jamais révéler de secret, de configuration interne ni de donnée d’un autre utilisateur.

## 5. Codes HTTP

- `200` lecture ou action synchrone réussie.
- `201` ressource créée.
- `202` traitement asynchrone accepté.
- `204` action réussie sans corps.
- `400` requête invalide.
- `401` authentification absente ou expirée.
- `403` action interdite.
- `404` ressource non visible ou inexistante.
- `409` conflit métier ou idempotence incompatible.
- `422` règle métier non satisfaite.
- `429` limite de débit atteinte.
- `500` erreur interne non exposée.
- `502` ou `503` partenaire indisponible.

## 6. Idempotence

Une même clé, utilisée par le même acteur sur la même opération avec le même contenu, retourne le résultat initial. Une réutilisation avec un contenu différent retourne `409 IDEMPOTENCY_CONFLICT`. La durée de conservation dépend du type d’opération et ne peut être inférieure à la fenêtre de rejeu du partenaire concerné.

## 7. Traitements asynchrones

Les opérations longues retournent un identifiant de tâche ou de transaction et un état initial. Les états autorisés sont définis dans les contrats partagés. Les clients suivent l’évolution par interrogation bornée, notification ou webhook signé.

## 8. Webhooks

- Signature HMAC ou asymétrique selon le partenaire.
- Horodatage et identifiant unique d’événement.
- Vérification de la fenêtre temporelle pour empêcher le rejeu.
- Livraison au moins une fois ; le destinataire doit être idempotent.
- Réessais bornés avec temporisation croissante et file d’échec.
- Possibilité de rejouer manuellement un événement depuis l’administration avec audit.

## 9. Pagination et filtres

Les filtres sont explicites et documentés. Les champs de tri sont limités à une liste blanche. Les limites maximales empêchent les lectures massives non contrôlées.

## 10. Critères d’acceptation

- Documentation OpenAPI générée depuis le code.
- Schémas de réponse et codes d’erreur testés.
- Toutes les mutations financières couvertes par l’idempotence.
- Identifiant de corrélation présent dans les réponses et journaux.
- Aucun détail technique sensible exposé par les erreurs de production.
