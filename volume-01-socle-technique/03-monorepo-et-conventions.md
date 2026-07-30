# Volume 1 — Monorepo et conventions

## 1. Gestionnaire et versions

Le dépôt utilise Node.js 22 LTS minimum, pnpm 10 et TypeScript strict. La version du gestionnaire est déclarée dans le `package.json` racine afin de rendre les installations reproductibles avec Corepack.

## 2. Structure

```text
apps/       applications déployables
packages/   bibliothèques partagées
infra/      infrastructure déclarative
scripts/    automatisation locale et CI
docs/       documentation technique proche du code
```

Chaque application possède son propre manifeste, ses scripts et ses tests. Une application ne doit jamais importer le code source interne d’une autre application.

## 3. Dépendances autorisées

- Les applications peuvent dépendre de `packages/contracts`, `packages/domain`, `packages/config` et, lorsque pertinent, `packages/ui`.
- `packages/domain` ne dépend d’aucun framework web, mobile ou ORM.
- `packages/contracts` ne contient ni accès aux données ni règle métier mutable.
- Les adaptateurs techniques dépendent des interfaces du domaine, jamais l’inverse.
- Les cycles de dépendances sont interdits.

## 4. Conventions TypeScript

- Mode `strict` obligatoire.
- Pas de `any` implicite.
- Exports publics centralisés par paquet.
- Types de données externes validés à l’entrée.
- Dates échangées au format ISO 8601 UTC.
- Montants échangés en unités mineures entières accompagnées d’un code devise ISO 4217.
- Identifiants opaques ; aucun sens métier ne doit être déduit de leur format.

## 5. Conventions Git

Les changements sont petits, cohérents et accompagnés d’un message de type Conventional Commits :

- `feat:` fonctionnalité ;
- `fix:` correction ;
- `docs:` documentation ;
- `test:` tests ;
- `refactor:` restructuration sans changement fonctionnel ;
- `build:` chaîne de construction ;
- `ci:` automatisation GitHub Actions ;
- `chore:` maintenance.

La branche principale reste déployable. Une fusion exige lint, typecheck, tests et build réussis.

## 6. Qualité minimale d’un paquet

Tout paquet exécutable doit fournir au minimum :

```json
{
  "scripts": {
    "build": "...",
    "lint": "...",
    "typecheck": "...",
    "test": "..."
  }
}
```

Un paquet sans tests doit néanmoins posséder une commande de test explicite et documentée, sans masquer indéfiniment l’absence de couverture.

## 7. Configuration et secrets

Les fichiers `.env` réels ne sont jamais versionnés. `.env.example` décrit uniquement les noms de variables, leur rôle et des valeurs factices sûres. Les valeurs de production proviennent d’un gestionnaire de secrets avec rotation, contrôle d’accès et audit.

## 8. Définition de terminé

Un lot est terminé lorsque :

1. le code et la documentation sont cohérents ;
2. les contrats publics sont versionnés ;
3. les tests pertinents existent ;
4. les commandes racine passent depuis une installation propre ;
5. aucune donnée sensible n’est introduite ;
6. les migrations et procédures de retour arrière sont documentées lorsqu’elles s’appliquent.
