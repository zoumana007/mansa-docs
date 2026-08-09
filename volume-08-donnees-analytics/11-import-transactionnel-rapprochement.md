# Import transactionnel des lots de rapprochement

## Objet

Cette note décrit la tranche exécutable qui suit la création des modèles PostgreSQL `ReconciliationBatch` et `ReconciliationItem`. Elle fixe le comportement du repository Prisma chargé d'importer un lot fournisseur sans dupliquer silencieusement une source déjà traitée.

## Référence de code

Implémentation :

`mansa-platform/apps/api-gateway/src/reconciliation/reconciliation.repository.ts`

Le repository s'appuie sur `PrismaService` et conserve toute la création d'un lot et de ses items dans une seule transaction PostgreSQL.

## Contrat d'import

Un import doit fournir :

- `providerId` non vide ;
- `sourceFingerprint` non vide ;
- période `periodStart` / `periodEnd` cohérente ;
- zéro ou plusieurs items normalisés par l'adaptateur fournisseur ;
- au moins une référence interne ou fournisseur pour chaque item ;
- devise ISO à trois lettres ;
- `providerOccurrenceCount >= 1`.

Les montants persistés utilisent `BIGINT` via Prisma et ne passent pas par des nombres flottants.

## Idempotence de source

Avant toute création, le repository cherche un lot par la clé persistée :

`(providerId, sourceFingerprint)`.

Si un lot existe déjà, l'import ne recrée ni lot ni item. Le résultat retourne le lot existant avec `reused = true`.

L'empreinte doit être calculée par l'adaptateur d'import sur une représentation canonique de la source. Une valeur aléatoire ou dépendante de l'heure annulerait la propriété d'idempotence et est interdite.

## Transaction de création

Pour une nouvelle source :

1. création du lot en statut `PROCESSING` ;
2. insertion des items avec `createMany` ;
3. calcul des compteurs `matchedItems` et `mismatchedItems` ;
4. mise à jour du lot ;
5. statut final `COMPLETED` ou `COMPLETED_WITH_MISMATCHES` ;
6. écriture de `completedAt` ;
7. commit unique de la transaction.

Une erreur avant le commit doit annuler l'ensemble du lot et de ses items.

## Normalisation

À la frontière repository :

- le fournisseur et les références sont nettoyés des espaces périphériques ;
- la devise est convertie en majuscules ;
- les statuts interne/fournisseur sont convertis en majuscules ;
- les champs optionnels absents ne sont pas écrits artificiellement avec `undefined`.

La normalisation métier plus complexe reste la responsabilité de l'adaptateur fournisseur avant comparaison.

## Compteurs matérialisés

Les compteurs du lot sont calculés à partir des items reçus dans la même opération :

- `totalItems` ;
- `matchedItems` ;
- `mismatchedItems`.

Les statuts `MISMATCHED` et `PARTIALLY_MATCHED` alimentent le compteur des écarts. Les futures résolutions devront mettre à jour `resolvedItems` et `ignoredItems` transactionnellement avec l'item concerné.

## Lecture bornée

La première lecture repository des items borne `take` entre 1 et 500 afin d'éviter une récupération non limitée. La pagination keyset complète devra être ajoutée avant exposition de volumes importants via API.

## Sécurité

Le repository n'expose aucune route publique et n'embarque aucun secret fournisseur. Les fichiers bruts ou identifiants d'accès doivent rester dans des services de stockage/secrets dédiés. Les contrôleurs futurs devront appliquer authentification, permission dédiée, tenant et corrélation avant de déléguer au repository.

## Critères d'acceptation de cette tranche

- une source nouvelle crée exactement un lot ;
- les items sont créés dans la même transaction ;
- les compteurs finaux correspondent aux statuts des items ;
- un lot avec au moins un écart finit `COMPLETED_WITH_MISMATCHES` ;
- un lot sans écart finit `COMPLETED` ;
- une source identique pour le même fournisseur réutilise le lot existant ;
- une période invalide est rejetée avant écriture ;
- un item sans référence est rejeté ;
- une occurrence fournisseur inférieure à 1 est rejetée ;
- la lecture des items reste bornée.

## Prochaine tranche

Le prochain lot doit introduire un fournisseur fictif tabulaire, transformer ses lignes en snapshots de comparaison, appeler le moteur pur `compareReconciliationTransactions`, importer le résultat via ce repository et ajouter des tests PostgreSQL d'idempotence et de rollback. Les routes de consultation seront branchées ensuite derrière les protections internes prévues par le contrat API.
