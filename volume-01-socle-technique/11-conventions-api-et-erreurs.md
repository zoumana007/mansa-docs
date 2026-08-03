# Conventions API et gestion des erreurs

## Objectif

Ce document fixe les conventions communes à toutes les API exposées par Mansa. Elles s’appliquent à l’API Gateway, aux services internes, aux webhooks et aux applications consommatrices.

## Versionnement

- Les routes publiques sont préfixées par `/api/v1`.
- Une rupture de contrat exige une nouvelle version majeure.
- Les ajouts compatibles restent dans la version courante.
- Les contrats partagés sont définis dans `packages/contracts`.

## Format des requêtes

- JSON UTF-8 pour les API HTTP.
- Dates au format ISO 8601 en UTC.
- Identifiants opaques de type UUID ou ULID.
- Montants financiers en unités mineures entières, accompagnés du code devise ISO 4217.
- Aucun montant financier en nombre flottant.

Exemple :

```json
{
  "amountMinor": 125000,
  "currency": "XOF"
}
```

## Enveloppe de succès

```json
{
  "data": {},
  "meta": {
    "requestId": "req_01...",
    "timestamp": "2026-08-03T11:35:00Z"
  }
}
```

Pour les listes paginées :

```json
{
  "data": [],
  "meta": {
    "requestId": "req_01...",
    "page": 1,
    "pageSize": 20,
    "total": 0,
    "hasNextPage": false
  }
}
```

## Enveloppe d’erreur

```json
{
  "error": {
    "code": "PAYMENT_INSUFFICIENT_FUNDS",
    "message": "Solde insuffisant",
    "details": {},
    "retryable": false
  },
  "meta": {
    "requestId": "req_01...",
    "timestamp": "2026-08-03T11:35:00Z"
  }
}
```

Le champ `message` peut être localisé côté application. La logique métier doit dépendre de `code`, jamais du texte.

## Catégories de codes d’erreur

- `AUTH_*` : authentification et session.
- `AUTHZ_*` : autorisations et permissions.
- `VALIDATION_*` : données invalides.
- `KYC_*` : identité et conformité.
- `ACCOUNT_*` : compte et portefeuille.
- `LEDGER_*` : grand livre.
- `PAYMENT_*` : paiement et encaissement.
- `TRANSFER_*` : transfert.
- `CARD_*` : cartes.
- `PARTNER_*` : intégrations externes.
- `RATE_LIMIT_*` : limitation de débit.
- `SYSTEM_*` : erreur technique interne.

## Statuts HTTP

- `200` : succès.
- `201` : ressource créée.
- `202` : traitement asynchrone accepté.
- `204` : succès sans contenu.
- `400` : requête invalide.
- `401` : authentification absente ou invalide.
- `403` : action interdite.
- `404` : ressource introuvable.
- `409` : conflit métier ou idempotence.
- `422` : validation métier impossible.
- `429` : limite de débit dépassée.
- `500` : erreur interne non exposée en détail.
- `502` ou `503` : dépendance externe indisponible.

## Idempotence

Les opérations financières, les créations de paiement et les webhooks utilisent l’en-tête `Idempotency-Key`.

Règles :

1. La clé est unique par acteur, opération et environnement.
2. La même clé avec le même contenu retourne le résultat initial.
3. La même clé avec un contenu différent retourne `409 IDEMPOTENCY_CONFLICT`.
4. Le stockage de la clé inclut l’empreinte de la requête, le résultat et la durée de rétention.
5. Une opération en cours retourne un statut explicite au lieu de créer un doublon.

## Corrélation et traçabilité

- Chaque requête reçoit un `requestId`.
- Les appels internes propagent `traceparent` et `requestId`.
- Les journaux ne contiennent ni secret, ni PIN, ni numéro complet de carte, ni document KYC brut.
- Les actions sensibles incluent l’acteur, le rôle, la ressource, l’horodatage et le résultat dans le journal d’audit.

## Sécurité

- Authentification par jeton court et mécanisme de renouvellement sécurisé.
- Validation stricte des entrées avant le domaine.
- Autorisation RBAC et, si nécessaire, ABAC.
- Limitation de débit par client, IP, utilisateur et type d’opération.
- Masquage systématique des données sensibles.
- Aucune erreur interne ou trace technique détaillée n’est renvoyée en production.

## Webhooks

- Signature obligatoire avec secret tournant.
- Horodatage vérifié pour empêcher la rejeu.
- Livraison au moins une fois : les consommateurs doivent être idempotents.
- Réponse rapide en `2xx`, traitement métier asynchrone.
- Politique de relance avec temporisation exponentielle et file d’échec.

## Critères d’acceptation

- Tous les contrôleurs utilisent les mêmes enveloppes.
- Tous les codes d’erreur sont centralisés et testés.
- Les opérations financières refusent l’absence de clé d’idempotence.
- Les identifiants de corrélation apparaissent dans les réponses et les logs.
- Les tests de contrat vérifient la compatibilité entre API et applications.
