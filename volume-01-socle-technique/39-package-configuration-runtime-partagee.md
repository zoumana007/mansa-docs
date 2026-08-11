# 39 — Package de configuration runtime partagée

## Objet

Cette tranche formalise le package partagé `@mansa/config` du monorepo `mansa-platform`.

L’objectif est de centraliser la validation des paramètres runtime non secrets utilisés par plusieurs produits Mansa, sans coupler le package à Node.js, React Native, Next.js ou un fournisseur d’infrastructure particulier.

## Principes

Le package doit :

- accepter une entrée explicite fournie par l’application consommatrice ;
- ne jamais lire directement `process.env` ;
- ne jamais stocker de secret ;
- normaliser les valeurs avant usage ;
- échouer au démarrage lorsque la configuration est invalide ;
- retourner une configuration immuable ;
- rester réutilisable côté API, Web, mobile, workers et outils internes.

## Contrat initial

La fonction `parseRuntimeConfig()` valide :

- `environment` : `demo`, `staging` ou `production` ;
- `countryCode` : code ISO 3166-1 alpha-2 ;
- `currency` : code ISO 4217 alpha-3 ;
- `apiBaseUrl` : URL HTTPS hors développement local ;
- `logLevel` : `debug`, `info`, `warn` ou `error`.

`logLevel` utilise `info` comme valeur par défaut. Les autres champs sont obligatoires.

## Sécurité réseau

Une URL HTTP en clair est refusée sauf pour :

- `localhost` ;
- `127.0.0.1`.

Cette exception existe uniquement pour le développement local. Les environnements distribués doivent utiliser HTTPS.

## Secrets

Ce package ne doit pas recevoir ni valider directement :

- clés API ;
- secrets JWT ;
- mots de passe ;
- clés privées ;
- tokens opérateurs ;
- credentials bancaires ;
- données HSM/KMS.

Ces valeurs doivent rester gérées par les mécanismes de secrets propres à l’environnement d’exécution.

## Intégration monorepo

Le package est situé dans :

```text
packages/config
```

Il suit les conventions existantes :

- TypeScript strict ;
- modules ESM ;
- compilation vers `dist` ;
- déclarations TypeScript exportées ;
- scripts `build`, `lint` et `typecheck` ;
- TypeScript fourni via le catalogue PNPM du workspace.

Aucune nouvelle dépendance runtime externe n’est introduite.

## Utilisation attendue

Chaque produit reste responsable de traduire sa source de configuration vers `RuntimeConfigInput`.

Exemples :

- API Gateway : variables d’environnement ou secret manager ;
- Next.js : variables publiques/serveur explicitement séparées ;
- React Native : configuration de build non secrète ;
- workers : environnement du runtime ;
- outils internes : paramètres de ligne de commande ou configuration dédiée.

Une application ne doit initialiser ses clients réseau qu’après validation réussie.

## Évolution

Les prochaines extensions doivent rester conservatrices. Ajouter un champ uniquement lorsqu’il est réellement partagé par plusieurs produits. Les configurations propres à un domaine métier ou à un fournisseur externe doivent rester dans leur module/adaptateur dédié afin d’éviter de transformer `@mansa/config` en registre global incontrôlé.

## Cohérence avec la plateforme

Cette tranche comble l’un des packages explicitement requis par le registre produit :

```text
packages/config
```

Elle ne modifie aucun flux financier et n’introduit aucune décision métier.

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

Compléter le second package partagé encore absent du registre, `packages/ui`, avec des tokens de design réellement cross-platform et sans imposer une dépendance React aux consommateurs qui n’en ont pas besoin.
