# Volume 1 — Registre des produits et traçabilité

## 1. Source de vérité

Le dépôt `mansa-docs` décrit les exigences fonctionnelles, les règles métier et les critères d’acceptation. Le dépôt `mansa-platform` contient l’implémentation et un registre technique lisible par machine dans `docs/product-registry.json`.

Le registre technique doit rester cohérent avec les volumes documentaires. Il ne remplace pas les spécifications détaillées, mais permet à la CI et aux outils de développement de détecter les suppressions accidentelles, les chemins incohérents et les écarts de périmètre.

## 2. Produits obligatoires

| Identifiant | Chemin plateforme | Volume principal | Type |
|---|---|---|---|
| `mobile-client` | `apps/mobile-client` | Volume 2 | Mobile |
| `mobile-merchant` | `apps/mobile-merchant` | Volume 3 | Mobile |
| `tpe-android` | `apps/tpe-android` | Volume 4 | Terminal Android |
| `mobile-admin-lite` | `apps/mobile-admin-lite` | Volume 5 | Mobile |
| `mobile-directory` | `apps/mobile-directory` | Volumes 2 et 3 | Mobile |
| `public-web` | `apps/public-web` | Volumes 2 et 10 | Web public |
| `business-web` | `apps/business-web` | Volumes 3 et 4 | Web professionnel |
| `admin-web` | `apps/admin-web` | Volume 5 | Web administratif |
| `api-gateway` | `apps/api-gateway` | Volumes 1 à 10 | Backend |
| `ai-services` | `apps/ai-services` | Volume 7 | Service isolé |
| `workers` | `apps/workers` | Volumes 1, 8 et 9 | Traitements asynchrones |

## 3. Paquets partagés obligatoires

- `packages/config` : conventions TypeScript, lint et outils.
- `packages/contracts` : DTO, schémas, événements et contrats partagés.
- `packages/domain` : primitives métier indépendantes des frameworks.
- `packages/ui` : design system et composants réutilisables.
- `packages/security` : politiques d’autorisation et primitives de sécurité.
- `packages/observability` : logs, métriques, traces et corrélation.

## 4. Règles de synchronisation

1. Toute création, suppression ou modification de chemin produit doit mettre à jour le registre et cette matrice.
2. Un produit obligatoire ne peut pas être retiré parce que son implémentation est incomplète.
3. Les chemins du registre doivent exister dans le monorepo ou être créés dans le même lot de travail.
4. Le fichier `docs/product-registry.schema.json` définit la forme attendue du registre.
5. La commande `pnpm registry:check` doit échouer en cas de registre illisible, d’identifiants dupliqués, de chemins dupliqués ou de valeurs obligatoires absentes.
6. Les évolutions incompatibles du registre nécessitent une augmentation de version et une note de migration.

## 5. Traçabilité exigence → code → test

Chaque module livré doit pouvoir être relié à :

- une section documentaire ;
- un chemin de code ;
- un contrat ou une interface publique ;
- au moins un test automatisé ;
- un critère de recette ;
- une décision d’architecture lorsque le choix n’est pas évident.

Le format minimal recommandé dans les pull requests est :

```text
Exigence : volume-XX/...#section
Code : apps/... ou packages/...
Tests : chemin du test ou commande de validation
Risque : faible, moyen ou élevé
Configuration manuelle : aucune ou liste précise
```

## 6. Critères d’acceptation

- Le registre est un JSON valide et respecte son schéma.
- Tous les identifiants et chemins sont uniques.
- Les onze produits obligatoires sont présents.
- Les six paquets partagés sont présents.
- La documentation et le registre utilisent les mêmes noms.
- Aucun secret, jeton, identifiant de production ou donnée personnelle n’est stocké dans le registre.
