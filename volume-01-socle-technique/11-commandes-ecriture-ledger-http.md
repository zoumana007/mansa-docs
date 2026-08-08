# 11 — Commandes d’écriture du ledger via HTTP

## Objectif

Cette spécification complète le contrat interne du grand livre pour la phase d’écriture. Elle fixe le format HTTP réellement sérialisable, les validations à appliquer avant toute transaction PostgreSQL et les invariants qui doivent rester identiques entre documentation et code.

## Pourquoi un format de transport dédié

Le domaine Mansa représente les montants en unités mineures entières avec `bigint`. Un `bigint` JavaScript n’est pas sérialisable nativement en JSON. Les routes HTTP ne doivent donc jamais accepter un nombre JSON pour `amountMinor` lorsqu’un dépassement de la plage sûre de JavaScript est possible.

Règle obligatoire :

- dans le domaine et la persistance, `amountMinor` est un entier exact ;
- sur HTTP JSON, `amountMinor` est une chaîne décimale positive, par exemple `"150000"` ;
- la conversion en `bigint` a lieu uniquement après validation du corps de requête ;
- aucun montant financier n’est converti en `number` ou en flottant.

## Route cible

```http
POST /v1/internal/ledger/transactions
```

La route est interne et doit être protégée par le mécanisme d’authentification de service déjà appliqué aux lectures ledger.

## Corps JSON

```json
{
  "reference": "PAYMENT:demo:001",
  "transactionType": "PAYMENT_CAPTURE",
  "idempotencyKey": "idem-demo-001",
  "correlationId": "corr-demo-001",
  "countryCode": "ML",
  "occurredAt": "2026-08-08T09:00:00.000Z",
  "metadata": {
    "source": "payment-service"
  },
  "entries": [
    {
      "accountId": "11111111-1111-4111-8111-111111111111",
      "direction": "DEBIT",
      "amountMinor": "1000",
      "currency": "XOF"
    },
    {
      "accountId": "22222222-2222-4222-8222-222222222222",
      "direction": "CREDIT",
      "amountMinor": "1000",
      "currency": "XOF"
    }
  ]
}
```

## Validations avant persistance

Le gateway doit refuser la requête avant toute écriture si une des conditions suivantes échoue :

1. le corps n’est pas un objet ;
2. `reference` est vide ;
3. `transactionType` est vide ;
4. `idempotencyKey` contient moins de huit caractères ;
5. `correlationId` est vide ;
6. `countryCode` n’est pas un code alpha-2 en majuscules ;
7. `occurredAt` n’est pas une date ISO-8601 valide ;
8. `entries` contient moins de deux écritures ;
9. un `accountId` n’est pas un UUID v4 valide ;
10. une direction n’est ni `DEBIT` ni `CREDIT` ;
11. `amountMinor` n’est pas une chaîne entière strictement positive ;
12. la devise n’est pas codée sur trois lettres majuscules ;
13. plusieurs devises sont présentes dans un même journal ;
14. le total des débits est différent du total des crédits ;
15. `metadata`, lorsqu’il est présent, contient autre chose que des valeurs chaîne.

## Idempotence

La clé d’idempotence est obligatoire et unique dans `LedgerTransaction`.

Comportement cible :

- première requête valide : création atomique ;
- même clé avec contenu métier identique : retourner la transaction déjà créée sans double écriture ;
- même clé avec contenu différent : erreur de conflit et aucune écriture ;
- un `requestFingerprint` persistant permet de distinguer répétition légitime et réutilisation incorrecte de clé.

Le fingerprint doit être calculé sur une représentation canonique des champs métier. Les secrets et en-têtes d’authentification ne doivent jamais y être inclus.

## Transaction PostgreSQL cible

La publication d’un journal doit être atomique. Une seule transaction PostgreSQL doit couvrir au minimum :

1. contrôle d’idempotence ;
2. lecture et validation des comptes concernés ;
3. création de `LedgerTransaction` ;
4. création ordonnée des `LedgerEntry` ;
5. mise à jour cohérente des projections nécessaires ;
6. création d’un événement `OutboxEvent` en statut `PENDING` ;
7. passage de la transaction ledger au statut `POSTED` ;
8. commit unique.

Si une étape échoue, aucune écriture partielle ne doit rester visible.

## Concurrence

Les scénarios de concurrence doivent couvrir :

- deux requêtes simultanées avec la même clé d’idempotence ;
- deux écritures simultanées sur le même compte ;
- publication pendant une compensation ;
- redémarrage après insertion de la transaction mais avant publication de l’événement ;
- répétition après timeout réseau côté appelant.

Le comportement doit être déterministe et vérifiable par tests PostgreSQL réels.

## Compensation

Une transaction `POSTED` n’est jamais modifiée ni supprimée. Une correction crée une transaction distincte liée à l’originale, avec les directions inversées et une nouvelle clé d’idempotence. Les champs de relation `reversalOfTransactionId` et `reversedByTransactionId` servent à préserver la chaîne d’audit.

## État d’implémentation

Le dépôt `mansa-platform` possède désormais une première validation de transport JSON dans :

```text
apps/api-gateway/src/ledger-write.validation.ts
```

Elle normalise les montants en `bigint` après validation et contrôle les invariants élémentaires de partie double avant accès à Prisma. Les tests associés se trouvent dans :

```text
apps/api-gateway/test/ledger-write.validation.test.mjs
```

Restent à implémenter dans les prochains lots : le service d’écriture Prisma atomique, le fingerprint d’idempotence, la route POST du contrôleur, la mise à jour des projections, l’outbox transactionnelle, la compensation et les tests d’intégration PostgreSQL/concurrence.

## Critères de recette de ce lot

- aucun montant HTTP n’est accepté comme nombre JSON ;
- les montants sont convertis en `bigint` seulement après validation ;
- un journal déséquilibré est rejeté ;
- une transaction multi-devise est rejetée ;
- les identifiants de comptes invalides sont rejetés ;
- les métadonnées non textuelles sont rejetées ;
- aucun secret n’est ajouté au dépôt.
