# Volume 1 — Contrats API et événements

## 1. Objectif

Les contrats constituent la frontière stable entre applications, backend, partenaires et traitements asynchrones. Ils doivent rester indépendants des frameworks et être validables automatiquement.

## 2. Enveloppe API

Toute réponse réussie expose les données demandées et un identifiant de corrélation. Toute erreur expose un code stable, un message compréhensible, l’identifiant de corrélation et, uniquement lorsque cela est sûr, des détails de validation.

```json
{
  "data": {},
  "meta": { "requestId": "req_..." }
}
```

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "La requête est invalide",
    "requestId": "req_...",
    "details": []
  }
}
```

## 3. Idempotence

Les opérations financières et toute création susceptible d’être rejouée exigent un en-tête `Idempotency-Key`. Le serveur associe la clé à l’acteur, à la route et à une empreinte de la requête. Une même clé avec un contenu différent est rejetée.

## 4. Pagination

Les listes utilisent une pagination par curseur pour les flux volumineux. La réponse contient `items` et `nextCursor`. Les limites maximales sont imposées côté serveur.

## 5. Modèle monétaire

```ts
interface MoneyDto {
  amountMinor: string;
  currency: string;
}
```

Le montant est sérialisé en chaîne décimale entière afin d’éviter les pertes de précision dans les clients JavaScript. Les valeurs négatives ne sont acceptées que pour les contrats qui les autorisent explicitement.

## 6. États transactionnels communs

- `CREATED` : intention enregistrée ;
- `PENDING` : traitement en cours ou attente partenaire ;
- `AUTHORIZED` : fonds autorisés ou réservés ;
- `SUCCEEDED` : résultat final réussi ;
- `FAILED` : échec final ;
- `CANCELLED` : annulation avant finalisation ;
- `REVERSED` : opération compensée après finalisation.

Aucun changement d’état ne doit contourner la machine à états du domaine.

## 7. Événements

Chaque événement comprend :

- `eventId` unique ;
- `eventType` versionné ;
- `occurredAt` en UTC ;
- `aggregateType` et `aggregateId` ;
- `correlationId` et, si disponible, `causationId` ;
- `schemaVersion` ;
- charge utile minimale sans secret.

Les consommateurs doivent être idempotents. Un événement déjà traité ne provoque pas de second effet financier.

## 8. Webhooks partenaires

Les webhooks sortants sont signés, horodatés, rejouables depuis l’administration et livrés avec réessais bornés. Les réponses `2xx` valident la livraison. Après épuisement des tentatives, l’élément rejoint une file d’échec inspectable.

Les webhooks entrants vérifient signature, horodatage, nonce ou identifiant d’événement avant toute modification métier.

## 9. Compatibilité

L’ajout de champs optionnels est compatible. La suppression, le renommage ou le changement de sens d’un champ nécessite une nouvelle version. Les contrats dépréciés disposent d’une date de retrait et d’un guide de migration.

## 10. Contrôles CI

- compilation du paquet de contrats ;
- tests des validateurs ;
- détection des ruptures OpenAPI ;
- validation des exemples ;
- vérification de l’absence de données sensibles dans les schémas et journaux.
