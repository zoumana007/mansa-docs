# Volume 1 — Structure du monorepo

## 1. Objectif

Le dépôt `mansa-platform` regroupe les applications et paquets partagés de Mansa dans un monorepo pnpm. Cette organisation garantit des versions cohérentes, des contrôles centralisés et une évolution progressive sans coupler les interfaces au stockage.

## 2. Arborescence de référence

```text
apps/
  api-gateway/
  admin-web/
  public-web/
  mobile-client/
  mobile-merchant/
  mobile-admin-lite/
  mobile-directory/
  tpe-android/
packages/
  config/
  contracts/
  domain/
  ui/
```

Les dossiers sont créés lorsqu’un lot fonctionnel commence. Un dossier vide n’est pas conservé artificiellement.

## 3. Responsabilités

### `apps/api-gateway`

Expose l’API HTTP versionnée, authentifie les requêtes, applique les autorisations et orchestre les cas d’usage. Il ne doit pas contenir les règles financières fondamentales.

### `packages/domain`

Contient les objets valeur et règles métier indépendantes des frameworks : argent, devises, identifiants, grand livre, états et invariants transactionnels.

### `packages/contracts`

Contient les contrats stables partagés aux frontières : enveloppes API, erreurs publiques, événements versionnés, pagination et types d’identifiants sérialisés. Aucun contrat ne dépend de NestJS, React ou Prisma.

### `packages/config`

Centralise les configurations réutilisables des outils, sans valeur d’environnement ni secret.

### `packages/ui`

Réunit uniquement les primitives visuelles véritablement communes. Les parcours métier restent dans chaque application.

## 4. Règles de dépendance

- Une application peut dépendre de `contracts`, `domain`, `config` et éventuellement `ui`.
- `contracts` et `domain` ne dépendent d’aucune application.
- `domain` ne dépend pas de `contracts` pour préserver son autonomie.
- Aucun frontend n’accède directement à PostgreSQL ou Prisma.
- Les fournisseurs externes sont appelés derrière des ports et adaptateurs.
- Les imports profonds entre paquets sont interdits : chaque paquet expose une API publique par son fichier `index.ts`.

## 5. Commandes racine

```bash
pnpm install
pnpm format:check
pnpm lint
pnpm typecheck
pnpm test
pnpm build
```

Les scripts racine délèguent aux espaces de travail. Tout nouveau paquet exécutable doit fournir les scripts pertinents afin d’être inclus automatiquement dans la validation.

## 6. Convention de commits

Les messages suivent la forme `type(scope): description`, par exemple :

- `feat(domain): ajouter les règles du grand livre`
- `fix(api): valider les paramètres de requête`
- `docs(volume-01): décrire les contrats partagés`
- `ci: ajouter la validation du monorepo`

## 7. Critères de cohérence

- Les chemins documentés correspondent aux chemins réels.
- Chaque paquet possède un nom npm sous l’espace `@mansa`.
- TypeScript est configuré en mode strict.
- Les valeurs de configuration sensibles ne sont jamais commitées.
- Les contrats exposés sont versionnés avant toute rupture de compatibilité.
