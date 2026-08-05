# Publication des contrats partagés

## Objectif

Le package `@mansa/contracts` est la source commune des types métier, commandes, réponses, routes et constantes utilisés par les applications Mansa. Un contrat n'est considéré comme disponible que lorsqu'il est à la fois compilé et publiquement importable.

## Règles de publication

Chaque nouveau domaine partagé doit respecter les quatre étapes suivantes :

1. créer le fichier métier dans `packages/contracts/src` ;
2. créer le contrat de transport ou d'API lorsqu'une exposition HTTP existe ;
3. déclarer le sous-chemin correspondant dans `packages/contracts/package.json` ;
4. ajouter l'export racine dans `packages/contracts/src/index.ts` lorsque le contrat doit être accessible depuis `@mansa/contracts`.

La présence d'un fichier TypeScript seul ne suffit pas. Sans entrée dans `exports`, un consommateur ne peut pas utiliser un import de sous-chemin stable tel que :

```ts
import type { TransactionLimitApiContract } from '@mansa/contracts/transaction-limits-api';
```

## Convention des sous-chemins

- domaine métier : `@mansa/contracts/<domaine>` ;
- contrat HTTP : `@mansa/contracts/<domaine>-api` ;
- nom de fichier : minuscules avec tirets ;
- aucune dépendance vers une application ou un framework ;
- extensions internes ESM en `.js` dans les imports TypeScript compilés.

## Vérifications obligatoires

Pour chaque ajout ou modification :

```bash
pnpm --filter @mansa/contracts typecheck
pnpm --filter @mansa/contracts build
pnpm --filter @mansa/contracts test
```

La vérification doit également confirmer que :

- le chemin déclaré dans `package.json` correspond exactement au fichier produit dans `dist` ;
- les types référencés sont exportés par leur module source ;
- aucun secret, identifiant réel ou donnée personnelle n'est présent ;
- les montants utilisent des unités mineures entières ;
- les dates transportées par API sont des chaînes ISO 8601 ;
- les commandes modifiant un état financier prévoient idempotence, audit et autorisation au niveau du service appelant.

## État du domaine des plafonds

Le domaine des plafonds transactionnels est défini par :

- `packages/contracts/src/transaction-limits.ts` ;
- `packages/contracts/src/transaction-limits-api.ts` ;
- le sous-chemin `@mansa/contracts/transaction-limits` ;
- le sous-chemin `@mansa/contracts/transaction-limits-api`.

Les routes couvrent la consultation, la création, la modification, l'évaluation, le suivi de consommation, la suspension et la réactivation d'un plafond.

## Critères d'acceptation

- Un consommateur peut importer le domaine et son API sans chemin interne vers `src` ou `dist`.
- Le build TypeScript produit les déclarations `.d.ts` correspondantes.
- Une suppression ou un renommage d'export public est traité comme un changement incompatible.
- La documentation du domaine et les chemins exposés par le package restent synchronisés.
