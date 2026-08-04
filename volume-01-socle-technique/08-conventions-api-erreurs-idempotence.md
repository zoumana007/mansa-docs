# Conventions API — erreurs, corrélation et idempotence

## Objectif

Ce document fixe les règles communes applicables à toutes les API HTTP de Mansa. Les applications Client, Commerçant, TPE, Admin, les intégrations partenaires et les services publics doivent recevoir des réponses homogènes et traçables.

## En-têtes obligatoires

### Requête

- `X-Request-Id` : identifiant de corrélation fourni par le client lorsqu’il existe. Sinon, l’API en génère un.
- `Idempotency-Key` : obligatoire pour toute commande susceptible de créer un effet financier ou externe.
- `Authorization` : jeton d’accès lorsque la route n’est pas publique.
- `Content-Type: application/json` pour les corps JSON.

### Réponse

- `X-Request-Id` : même identifiant que celui utilisé dans les logs, traces et événements.
- `Retry-After` : présent en cas de limitation ou d’indisponibilité temporaire lorsque le délai est connu.

Aucune donnée sensible, aucun jeton, aucun PIN, aucun secret partenaire et aucune donnée KYC brute ne doit être copié dans les en-têtes de corrélation ou les journaux.

## Format d’erreur commun

Toutes les erreurs applicatives exposées utilisent le contrat `ApiErrorResponse` du paquet `@mansa/contracts`.

```json
{
  "code": "VALIDATION_FAILED",
  "message": "La requête contient des données invalides.",
  "requestId": "req_01H...",
  "timestamp": "2026-08-04T10:30:00.000Z",
  "details": [
    {
      "field": "amountMinor",
      "reason": "must_be_positive"
    }
  ]
}
```

Le message est destiné à l’utilisateur ou à l’intégrateur. Le champ `reason` est stable, exploitable par une interface et non localisé. Les informations techniques internes restent dans les logs sécurisés.

## Catalogue initial des codes

| Code | HTTP recommandé | Utilisation |
| --- | ---: | --- |
| `AUTHENTICATION_REQUIRED` | 401 | Session absente, expirée ou invalide. |
| `FORBIDDEN` | 403 | Identité connue mais autorisation insuffisante. |
| `VALIDATION_FAILED` | 400 | Corps, paramètres ou règle de saisie invalides. |
| `RESOURCE_NOT_FOUND` | 404 | Ressource inexistante ou volontairement masquée. |
| `CONFLICT` | 409 | État incompatible avec la commande demandée. |
| `IDEMPOTENCY_CONFLICT` | 409 | Même clé réutilisée avec une charge utile différente. |
| `RATE_LIMITED` | 429 | Limite de requêtes ou protection anti-abus atteinte. |
| `PARTNER_UNAVAILABLE` | 503 | Banque, Mobile Money, processeur ou service externe indisponible. |
| `INTERNAL_ERROR` | 500 | Erreur interne non exposée au client. |

Un nouveau code ne peut être ajouté que s’il est transversal, documenté et ajouté au contrat partagé. Les codes propres à un domaine doivent rester rares et suivre la même convention en majuscules avec séparateur `_`.

## Idempotence

### Routes concernées

L’idempotence est obligatoire au minimum pour :

- création de paiement ;
- transfert ;
- remboursement ;
- encaissement TPE ;
- paiement de taxe, amende ou frais de scolarité ;
- émission ou remplacement de carte ;
- création d’un décaissement ou règlement commerçant ;
- traitement d’un webhook partenaire.

### Règles

1. Une clé identifie une commande, un acteur et un périmètre métier.
2. Une répétition strictement identique retourne le résultat initial sans reproduire l’effet.
3. Une même clé avec un corps différent retourne `IDEMPOTENCY_CONFLICT`.
4. Le résultat est conservé pendant une durée configurable adaptée au risque du produit.
5. Les erreurs temporaires avant création d’effet peuvent être rejouées ; les effets confirmés ne le sont jamais.
6. La vérification de la clé et l’écriture de l’effet doivent être atomiques ou protégées par une contrainte équivalente.

## Corrélation et audit

Le `requestId` doit être propagé vers :

- l’API Gateway ;
- les services internes ;
- les workers ;
- les événements métier ;
- les appels partenaires lorsque le partenaire accepte une référence ;
- le journal d’audit et les métriques.

Une opération financière possède en plus une référence métier stable distincte du `requestId`. Le `requestId` suit une tentative technique ; la référence métier suit l’opération pendant tout son cycle de vie.

## Réessais

Les clients ne doivent réessayer automatiquement que les erreurs explicitement temporaires : limitation, délai dépassé, indisponibilité partenaire et certaines erreurs réseau. Les réessais utilisent la même clé d’idempotence, un délai exponentiel avec jitter et une limite stricte de tentatives.

Aucun réessai automatique ne doit être effectué sur `FORBIDDEN`, `VALIDATION_FAILED`, `CONFLICT` ou `IDEMPOTENCY_CONFLICT` sans modification préalable de la requête ou de l’état.

## Critères d’acceptation

- Toutes les routes exposent un `X-Request-Id`.
- Toutes les commandes financières refusent l’absence de clé d’idempotence.
- Deux requêtes identiques avec la même clé ne créent qu’un seul effet.
- Deux corps différents avec la même clé produisent `IDEMPOTENCY_CONFLICT`.
- Les erreurs ne divulguent ni stack trace, ni secret, ni donnée KYC brute.
- Les logs, traces, événements et audits permettent de reconstituer une opération à partir du `requestId` et de sa référence métier.
