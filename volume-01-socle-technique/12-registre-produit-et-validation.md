# Registre produit et validation automatisée

## Objectif

Le dépôt `mansa-platform` contient un registre lisible par machine dans `docs/product-registry.json`. Ce fichier maintient la liste canonique des applications, services, packages partagés et emplacements d’infrastructure prévus dans le monorepo.

Il complète la documentation fonctionnelle : les volumes décrivent le comportement attendu, tandis que le registre fixe les identifiants techniques et les chemins qui doivent rester cohérents pendant la construction.

## Contenu obligatoire

Chaque produit déclare :

- un `id` stable et unique ;
- un `type` ;
- son `path` cible dans le monorepo ;
- la technologie principale ;
- le booléen `required` indiquant s’il appartient au périmètre obligatoire.

Le registre contient aussi :

- `sharedPackages` pour les bibliothèques transverses ;
- `infrastructure` pour les dossiers d’exploitation et d’intégration continue ;
- `rules` pour les invariants techniques non négociables.

## Correspondance avec les volumes

| Identifiant technique | Documentation principale |
|---|---|
| `mobile-client` | Volume 02 — Application client |
| `mobile-merchant` et `business-web` | Volume 03 — Expérience commerçant |
| `tpe-android` | Volume 04 — Application TPE |
| `admin-web` et `mobile-admin-lite` | Volume 05 — Administration |
| `mobile-directory` | Spécification Annuaire Hub et parcours associés |
| `api-gateway` | Volume 01 — Socle technique |
| `ai-services` | Volume 07 — Intelligence artificielle |
| `workers` | Volumes 01, 08 et 09 selon les traitements |
| `public-web` | Site public, présentation produit et acquisition |

## Validation

La commande suivante vérifie le format et les invariants du registre :

```bash
pnpm registry:check
```

Elle contrôle notamment :

1. la présence des champs obligatoires ;
2. l’unicité des identifiants et chemins produit ;
3. le type booléen de `required` et des règles ;
4. l’absence de doublons dans les packages partagés et entrées d’infrastructure ;
5. la validité syntaxique du JSON.

La commande globale `pnpm validate` exécute ce contrôle avant le formatage, le lint, le typage, les tests et le build. Le workflow GitHub Actions exécute également `pnpm registry:check` après l’installation des dépendances.

## Règles d’évolution

- Ne pas renommer un identifiant publié sans décision d’architecture et plan de migration.
- Ne pas supprimer une entrée parce que son squelette n’est pas encore créé.
- Ajouter toute nouvelle application ou service au registre et à la documentation dans le même lot de travail.
- Vérifier qu’un déplacement de dossier met à jour le registre, les scripts, le CI et les liens documentaires.
- Ne jamais placer de secret, URL privée, jeton ou donnée réelle dans le registre.

## Critères d’acceptation

- `docs/product-registry.json` est un JSON valide.
- `pnpm registry:check` se termine avec un code nul.
- Les identifiants et chemins sont uniques.
- Chaque produit obligatoire possède une documentation de référence.
- Toute divergence entre le registre et l’arborescence cible est traitée comme une dette explicite, et non masquée par la suppression d’une entrée.
