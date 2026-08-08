# Port de persistance des écritures du grand livre

## 1. Objectif

Ce lot introduit un port de persistance explicite entre le domaine ledger et le futur adaptateur PostgreSQL/Prisma. Le but est de figer les invariants de lecture avant d’introduire des types ou dépendances Prisma dans le code métier.

## 2. Fichiers ajoutés ou modifiés

Dans `mansa-platform` :

```text
packages/contracts/src/ledger-entry-repository.ts
packages/contracts/src/ledger-api.ts
packages/contracts/test/ledger-entry-repository.test.mjs
```

Le nouveau contrat est réexporté par `ledger-api.ts`.

## 3. Contrat du repository

Le port expose :

```text
LedgerEntryRepository.listEntries(query)
```

La requête de stockage contient :

- `accountId` obligatoire ;
- `from` et `to` optionnels ;
- `after` optionnel sous forme de position keyset déjà décodée ;
- `take` obligatoire.

Le repository retourne uniquement une collection ordonnée de `LedgerEntry`. La transformation en page API et la génération du curseur suivant restent hors du repository.

## 4. Séparation des responsabilités

Le contrat ne dépend d’aucun type Prisma.

La couche API/application reste responsable de :

1. valider la requête publique ;
2. décoder le curseur opaque ;
3. vérifier que le curseur appartient au compte demandé ;
4. demander `limit + 1` lignes au repository ;
5. construire la page finale et le prochain curseur.

Le repository reste responsable de :

1. filtrer par compte ;
2. appliquer les bornes temporelles ;
3. appliquer le prédicat keyset strictement après `(postedAt, entryId)` ;
4. ordonner par `postedAt ASC, entryId ASC` ;
5. limiter la lecture à `take` lignes.

## 5. Limite de lecture

La requête publique autorise actuellement `limit` jusqu’à 200. Le port autorise donc `take` jusqu’à 201 afin de supporter la stratégie `limit + 1` sans requête `COUNT` supplémentaire.

Cette limite empêche un consommateur interne de contourner involontairement la borne définie pour la pagination ledger.

## 6. Validation du port

`validateLedgerEntryRepositoryQuery` rejette :

- un `accountId` vide ;
- des dates invalides ;
- une plage temporelle inversée ;
- une position keyset invalide ;
- un `take` non entier, inférieur à 1 ou supérieur à 201.

Les codes d’erreur sont stables afin de pouvoir être testés et observés sans dépendre du texte des messages.

## 7. Tests ajoutés

Le test de contrat couvre :

- une requête valide utilisant une fenêtre maximale de 201 lignes ;
- le rejet d’une position keyset invalide ;
- le rejet d’un `take` supérieur à la borne ;
- le rejet d’une plage temporelle inversée ;
- la stabilité du garde de type des codes d’erreur.

## 8. Traduction attendue vers Prisma/PostgreSQL

Le futur adaptateur devra traduire `after` en prédicat SQL équivalent à :

```sql
posted_at > :postedAt
OR (posted_at = :postedAt AND id > :entryId)
```

avec :

```sql
WHERE account_id = :accountId
ORDER BY posted_at ASC, id ASC
LIMIT :take
```

et les bornes temporelles optionnelles.

L’index cible reste :

```text
(account_id, posted_at, id)
```

Le nom exact de l’index et sa migration doivent être définis lorsque le modèle Prisma ledger sera branché.

## 9. Sécurité et cohérence

Aucun secret, identifiant bancaire ou paramètre d’environnement n’est ajouté. Le port reste purement contractuel et ne donne aucun accès direct à une base de données.

## 10. Suite recommandée

Le prochain lot doit implémenter ce port dans l’infrastructure Prisma/PostgreSQL, puis ajouter des tests d’intégration qui vérifient :

- plusieurs pages avec collisions de `postedAt` ;
- l’index keyset ;
- les bornes `from` / `to` ;
- l’absence de doublon et d’omission ;
- le comportement lorsque de nouvelles écritures sont insérées entre deux lectures.
