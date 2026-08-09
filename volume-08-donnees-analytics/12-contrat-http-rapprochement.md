# Contrat HTTP interne du rapprochement financier

## Objet

Cette spécification fixe le comportement HTTP minimal du module de rapprochement financier exposé par l’API Gateway. Les routes sont internes et protégées par l’identité de service. Elles ne doivent pas être exposées directement aux applications publiques.

## Préfixe

Les routes sont versionnées sous `v1/internal/reconciliation`.

## Lecture des lots

`GET /batches`

Paramètres :

- `limit` facultatif, entier entre 1 et 100, défaut 50 ;
- `cursor` facultatif, opaque pour le client.

Réponse :

```json
{
  "data": [],
  "hasNextPage": false,
  "nextCursor": "opaque-si-présent"
}
```

Un curseur malformé renvoie `400`. Le client ne doit jamais tenter de décoder ou reconstruire le curseur.

## Lecture d’un lot

`GET /batches/:batchId`

- `batchId` est un UUID v4 ;
- `404` si le lot n’existe pas.

## Lecture des items d’un lot

`GET /batches/:batchId/items`

Paramètres :

- `limit` facultatif, entier entre 1 et 500, défaut 100 ;
- `cursor` facultatif, opaque ;
- `404` si le lot n’existe pas ;
- `400` si le curseur ou la limite est invalide.

La pagination est keyset et stable. Aucun numéro de page n’est exposé.

## Lecture d’un item

`GET /items/:itemId`

- `itemId` est un UUID v4 ;
- `404` si l’item n’existe pas.

## Résolution manuelle

`POST /items/:itemId/resolve`

Corps obligatoire :

```json
{
  "status": "RESOLVED",
  "resolutionNote": "Écart vérifié avec le fournisseur.",
  "reasonCode": "PROVIDER_CONFIRMED",
  "idempotencyKey": "resolve-item-uuid-v1",
  "correlationId": "corr-...",
  "actorId": "service-account-or-operator-id",
  "actorType": "SERVICE_ACCOUNT"
}
```

`status` accepte uniquement `RESOLVED` ou `IGNORED`.

La commande doit rester atomique : mutation de l’item, incrément du compteur du lot et écriture dans `OperationalAuditLog` appartiennent à la même transaction PostgreSQL. Une erreur d’audit annule la résolution.

L’idempotency key ne peut pas être réutilisée pour une résolution différente. Un replay strictement identique renvoie le résultat déjà persistant sans dupliquer l’audit ni les compteurs.

## Erreurs

- `400` : limite invalide, curseur invalide, statut invalide, champs d’audit manquants, item déjà résolu ou conflit d’idempotence ;
- `404` : lot ou item absent ;
- `401/403` : identité de service non autorisée ;
- `5xx` : panne technique ou violation inattendue, sans fuite d’information sensible.

## Validation automatisée

Le dépôt `mansa-platform` contient désormais :

- tests PostgreSQL de persistance, concurrence, pagination et résolution atomique ;
- tests du contrôleur interne pour les limites, curseurs, `404`, validation du statut et traduction des erreurs métier ;
- validation du passage complet des métadonnées d’audit vers le repository.

Les tests du contrôleur restent volontairement indépendants d’un serveur HTTP réel afin de valider rapidement le contrat applicatif. Une tranche ultérieure peut compléter ce niveau par des tests end-to-end NestJS avec identité de service attestée lorsque le mécanisme définitif d’authentification workload sera branché.

## Critères d’acceptation

1. Toutes les lectures sont bornées et paginées par curseur.
2. Aucun payload fournisseur brut n’est renvoyé par les routes de consultation.
3. Toute résolution est traçable, idempotente et atomique.
4. Les erreurs de validation sont déterministes et ne deviennent pas des `500`.
5. Les routes restent derrière `InternalServiceGuard` jusqu’au remplacement par l’identité workload définitive.
