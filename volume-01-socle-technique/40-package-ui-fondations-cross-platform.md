# 40 — Package UI : fondations cross-platform

## Objet

Cette tranche formalise le package partagé `@mansa/ui` du monorepo `mansa-platform`.

L’objectif est de fournir un socle visuel cohérent aux applications Web et mobiles sans imposer React, React Native, Next.js ou une bibliothèque de composants particulière aux consommateurs qui n’en ont pas besoin.

## Principes

Le package doit :

- rester sans dépendance runtime à un framework UI ;
- centraliser uniquement les fondations réellement communes ;
- exporter des valeurs immuables ;
- éviter de dupliquer les espacements, rayons et échelles typographiques dans chaque produit ;
- préserver l’accessibilité des contrôles interactifs ;
- séparer les fondations structurelles de la palette finale de marque ;
- rester utilisable par Web, React Native, TPE et outils de génération de design.

## Fondations initiales

Le package exporte les groupes suivants.

### Espacements

Une échelle partagée couvre les besoins depuis `0` jusqu’aux grands espacements de présentation.

Elle sert notamment à :

- paddings ;
- gaps ;
- marges ;
- espacement entre sections ;
- densité des cartes et panneaux.

### Rayons

L’échelle de rayons couvre :

- absence de rayon ;
- petits contrôles ;
- cartes ;
- panneaux ;
- surfaces fortement arrondies ;
- pills/badges.

### Typographie

Le package fournit une base commune pour :

- tailles de texte ;
- hauteurs de ligne ;
- poids de police.

Il ne fixe pas encore une famille de police globale. Ce choix reste lié au design system final et aux contraintes des plateformes.

### Contrôles

Les tailles de contrôles partagées incluent une cible tactile minimale de `44` pixels logiques.

Les produits peuvent utiliser des contrôles plus grands, mais ne doivent pas réduire les zones interactives critiques sous cette référence sans justification d’accessibilité spécifique.

### Motion

Le package expose uniquement des durées de référence :

- instantané ;
- rapide ;
- normal ;
- lent.

Les produits restent responsables de respecter `prefers-reduced-motion`, les réglages d’accessibilité natifs et les contraintes de performance.

## Couleurs sémantiques

Le package n’embarque pas une palette finale arbitraire.

Chaque produit fournit explicitement les rôles suivants :

- `background` ;
- `surface` ;
- `surfaceRaised` ;
- `text` ;
- `textMuted` ;
- `border` ;
- `primary` ;
- `onPrimary` ;
- `success` ;
- `warning` ;
- `danger`.

La fonction `createUiTheme()` :

- normalise les chaînes ;
- refuse les valeurs vides ;
- retourne un objet immuable ;
- réutilise les mêmes fondations structurelles pour tous les thèmes.

Cette séparation évite de figer prématurément des couleurs de marque différentes de celles du design source.

## Accessibilité

Les applications consommatrices doivent compléter ce socle par :

- tests de contraste ;
- tailles de texte dynamiques lorsque la plateforme le permet ;
- navigation clavier Web ;
- focus visible ;
- lecteurs d’écran ;
- réduction des animations ;
- tailles tactiles suffisantes ;
- états loading/error/disabled explicites.

Le package `@mansa/ui` fournit des fondations, pas une certification d’accessibilité automatique.

## Architecture

Le package est situé dans :

```text
packages/ui
```

Il utilise :

- TypeScript strict ;
- ESM ;
- sortie `dist` ;
- déclarations TypeScript ;
- aucune dépendance runtime externe ;
- tests Node sur la sortie compilée.

## Tests attendus

Les tests couvrent au minimum :

- immuabilité des objets exportés ;
- cible tactile minimale ;
- normalisation des rôles de couleur ;
- refus des valeurs sémantiques vides ;
- stabilité des références partagées dans un thème.

## Frontière avec les composants

Cette tranche ne crée pas encore de bibliothèque React/React Native complète.

Les composants visuels de haut niveau doivent être ajoutés ensuite dans des couches adaptées aux plateformes si nécessaire, en réutilisant les tokens de `@mansa/ui` plutôt qu’en dupliquant les valeurs.

Exemples :

- boutons ;
- champs ;
- cartes ;
- modales ;
- bottom sheets ;
- navigation ;
- tableaux Admin ;
- composants TPE.

## Cohérence avec le registre produit

Cette tranche matérialise le package requis :

```text
packages/ui
```

Après `packages/config` puis `packages/ui`, tous les chemins de packages partagés déclarés dans le registre doivent être audités non seulement pour leur présence, mais aussi pour leur contenu réellement exploitable.

## Validation attendue

Le dépôt `mansa-platform` doit continuer à passer :

- `pnpm registry:check` ;
- `pnpm contracts:check` ;
- `pnpm format:check` ;
- `pnpm lint` ;
- `pnpm typecheck` ;
- `pnpm test` ;
- `pnpm build`.

## Prochaine tranche recommandée

Auditer les autres packages partagés (`contracts`, `domain`, `security`, `observability`) et sélectionner le premier qui est encore un scaffold ou dont les tests/contrats ne correspondent pas au registre produit. La priorité doit rester un socle réellement consommable plutôt qu’une simple présence de répertoire.
