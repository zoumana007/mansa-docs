# Configuration d’exécution de l’API Gateway

## Objectif

L’API Gateway doit refuser toute configuration ambiguë avant d’ouvrir le port HTTP. La validation est centralisée dans `apps/api-gateway/src/runtime-config.ts` afin d’éviter des règles différentes entre le démarrage, les tests et les futurs workers.

## Variables supportées

| Variable | Valeur par défaut | Valeurs autorisées | Rôle |
| --- | --- | --- | --- |
| `NODE_ENV` | `development` | `development`, `test`, `staging`, `production` | Sélectionne l’environnement d’exécution. |
| `HOST` | `0.0.0.0` | Adresse d’écoute non vide | Définit l’interface réseau HTTP. |
| `PORT` | `3000` | Entier de `1` à `65535` | Définit le port HTTP. |
| `TRUST_PROXY` | `false` | `true` ou `false` | Active la confiance du proxy Express uniquement derrière un proxy inverse maîtrisé. |
| `DATABASE_URL` | aucune valeur de production | URL PostgreSQL | Utilisée par Prisma. Une valeur locale illustrative figure dans `.env.example`. |

## Règles de sécurité

- Aucun fichier `.env` contenant des secrets ne doit être versionné.
- `TRUST_PROXY=true` n’est autorisé que lorsque l’infrastructure réseau et les en-têtes transférés sont contrôlés.
- Une valeur invalide provoque l’arrêt immédiat du processus avant l’écoute réseau.
- Les environnements supplémentaires éventuels doivent faire l’objet d’une décision d’architecture avant d’être acceptés.

## Critères d’acceptation

1. Sans variable, l’API écoute sur `0.0.0.0:3000` en environnement `development`.
2. `PORT=3000abc`, `PORT=0` et `PORT=70000` sont refusés.
3. Une valeur `NODE_ENV` non reconnue est refusée.
4. `TRUST_PROXY` n’accepte que les chaînes exactes `true` et `false`.
5. Le démarrage utilise exclusivement la configuration validée.
6. Les cas nominaux et les rejets sont couverts par des tests automatisés.

## Commandes de validation

```bash
pnpm --filter @mansa/api-gateway test
pnpm --filter @mansa/api-gateway lint
pnpm --filter @mansa/api-gateway build
pnpm format:check
```

## Évolution prévue

Les prochaines extensions doivent ajouter, sans exposer de secrets, la validation des origines CORS, des limites de requêtes, des paramètres de journalisation et de l’accès PostgreSQL. Toute nouvelle variable doit être documentée ici et ajoutée à `.env.example`.
