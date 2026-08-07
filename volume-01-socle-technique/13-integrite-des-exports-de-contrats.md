# Intégrité des exports de contrats partagés

## Objectif

Le package `@mansa/contracts` constitue la frontière de types commune entre les applications, services et outils Mansa. Chaque sous-chemin déclaré dans son champ `exports` doit correspondre exactement à un fichier source TypeScript compilable afin d’éviter les contrats documentés mais impossibles à importer.

## Règles obligatoires

- Le point d’entrée `.` correspond à `packages/contracts/src/index.ts` et produit `dist/index.js` ainsi que `dist/index.d.ts`.
- Chaque sous-chemin `./nom` correspond à `packages/contracts/src/nom.ts`.
- Les cibles `types` et `import` doivent respecter le même nom de fichier que le sous-chemin public.
- Aucun sous-chemin public ne peut pointer vers un fichier source absent.
- Toute nouvelle API partagée doit être ajoutée au manifeste `packages/contracts/package.json` au même moment que son fichier source.
- Une modification d’un nom de contrat doit mettre à jour simultanément le fichier source, le manifeste d’exports, les consommateurs et la documentation associée.

## Validation automatisée

Le dépôt plateforme contient `scripts/validate-contract-exports.mjs`. Le script lit `packages/contracts/package.json`, contrôle chaque entrée publique, vérifie la correspondance des sorties JavaScript et déclarations TypeScript, puis confirme l’existence du fichier `src/*.ts` associé.

La commande racine suivante exécute ce contrôle :

```bash
pnpm contracts:check
```

Elle est également intégrée à `pnpm validate`, après la validation du registre produit et avant les contrôles de format, lint, types, tests et build.

## Critères d’acceptation

1. `pnpm contracts:check` termine avec un code de sortie `0` lorsque tous les sous-chemins publics sont cohérents.
2. Une entrée déclarant un fichier source absent provoque un échec explicite.
3. Une cible `types` ou `import` dont le nom ne correspond pas au sous-chemin provoque un échec explicite.
4. Aucun secret, identifiant partenaire ou donnée d’environnement n’est nécessaire à ce contrôle.
5. La validation est déterministe et exécutable localement comme en intégration continue.

## Évolution

Lorsque le package de contrats gagnera des points d’entrée volontairement internes ou des fichiers générés, ces exceptions devront être décrites explicitement dans la politique d’exports plutôt que contournées silencieusement dans le script.
