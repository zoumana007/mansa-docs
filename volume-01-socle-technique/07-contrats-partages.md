# Volume 1 — Contrats partagés

## 1. Rôle

Le paquet `packages/contracts` contient les structures stables échangées entre applications et backend. Il ne contient ni accès à la base, ni logique liée à NestJS, React Native, Next.js ou Android.

## 2. Règles de dépendance

- Les contrats peuvent dépendre uniquement de TypeScript et de bibliothèques de validation explicitement approuvées.
- Le domaine peut utiliser les primitives des contrats lorsque leur sens métier est stable.
- Les applications consomment les contrats par l’export public `@mansa/contracts`.
- Aucun module ne doit importer un fichier interne du paquet par un chemin profond.
- Toute rupture de forme nécessite une nouvelle version de contrat ou une migration coordonnée.

## 3. Montants

Un montant est représenté par :

- `amountMinor` : entier en unité mineure ;
- `currency` : code monétaire ISO pris en charge.

Les additions et soustractions entre devises différentes sont interdites. Aucun montant financier n’utilise de nombre flottant.

## 4. Idempotence

Les commandes financières utilisent une clé d’idempotence non vide et de longueur bornée. Une même clé associée à une charge différente produit une erreur `IDEMPOTENCY_CONFLICT`.

## 5. Transactions

Les références de transaction exposent un identifiant, un type, un statut et les dates utiles. Les statuts finaux sont déclarés dans une liste commune afin d’éviter des interprétations différentes entre les interfaces.

## 6. Erreurs API

Toutes les erreurs publiques suivent la forme :

```json
{
  "code": "VALIDATION_FAILED",
  "message": "La requête est invalide.",
  "requestId": "req_...",
  "timestamp": "2026-07-30T15:00:00.000Z",
  "details": [
    { "field": "amountMinor", "reason": "must be a positive integer" }
  ]
}
```

Le message destiné à l’utilisateur ne doit pas révéler de trace, de secret, de requête SQL ou de donnée interne. `requestId` permet la corrélation avec les journaux techniques.

Codes initiaux : authentification requise, accès interdit, validation échouée, ressource absente, conflit, conflit d’idempotence, limitation de débit, partenaire indisponible et erreur interne.

## 7. Pagination

Les listes utilisent une pagination par curseur :

- `cursor` est opaque pour le client ;
- `limit` vaut 20 par défaut et ne dépasse pas 100 ;
- la réponse contient `data`, `nextCursor` lorsque présent et `hasNextPage` ;
- un tri stable et déterministe est obligatoire.

La pagination par numéro de page est réservée aux exports ou écrans administratifs ne parcourant pas des données transactionnelles changeantes.

## 8. Validation de cohérence

Le paquet doit réussir `lint`, `typecheck`, `test` et `build` depuis la racine. Toute nouvelle structure partagée doit être exportée depuis `packages/contracts/src/index.ts` et documentée ici ou dans le volume fonctionnel concerné.
