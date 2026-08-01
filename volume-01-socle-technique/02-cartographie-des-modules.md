# Cartographie des modules Mansa

## Objectif

Cette cartographie définit le périmètre minimal obligatoire du monorepo `mansa-platform`. Elle sert de référence commune pour la documentation, le code, les tests, la CI et les décisions d’architecture.

## Applications

| Identifiant | Chemin cible | Technologie cible | Responsabilité principale |
|---|---|---|---|
| `mobile-client` | `apps/mobile-client` | React Native | Comptes, paiements, transferts, cartes, budgets, coffre, support et Jini. |
| `mobile-merchant` | `apps/mobile-merchant` | React Native | Encaissement, catalogue, équipe, factures, fidélité, promotions et pilotage du commerce. |
| `mobile-admin-lite` | `apps/mobile-admin-lite` | React Native | Supervision mobile, alertes, validations et actions administratives limitées. |
| `mobile-directory` | `apps/mobile-directory` | React Native | Annuaire, géolocalisation, filtres sectoriels, profils et mini-sites. |
| `tpe-android` | `apps/tpe-android` | Kotlin/Android | Paiement sur terminal, carte, QR, mode hors ligne contrôlé et intégration matérielle. |
| `admin-web` | `apps/admin-web` | Next.js | Configuration globale, conformité, risque, audit, support, partenaires et reporting. |
| `public-web` | `apps/public-web` | Next.js | Présentation officielle, chiffres publics, téléchargement et informations institutionnelles. |
| `business-web` | `apps/business-web` | Next.js | Offre commerçants, partenaires, terminaux, documentation et souscription professionnelle. |
| `api-gateway` | `apps/api-gateway` | NestJS | API publique et interne, authentification, orchestration et exposition des contrats. |
| `ai-services` | `apps/ai-services` | Service isolé | Jini, support assisté, détection de fraude et recommandations contrôlées. |
| `workers` | `apps/workers` | Node.js | Événements, notifications, webhooks, traitements différés et tâches planifiées. |

## Paquets partagés

| Identifiant | Chemin cible | Contenu autorisé |
|---|---|---|
| `config` | `packages/config` | Configurations TypeScript, lint, tests et compilation. |
| `contracts` | `packages/contracts` | DTO, schémas, événements, codes d’erreur et contrats versionnés. |
| `domain` | `packages/domain` | Entités, value objects, règles métier et ports indépendants des frameworks. |
| `ui` | `packages/ui` | Jetons de design et composants partagés compatibles avec les surfaces concernées. |
| `security` | `packages/security` | Permissions, politiques, contrôles d’accès et primitives de sécurité. |
| `observability` | `packages/observability` | Journalisation structurée, métriques, traces et corrélation. |

## Domaines backend obligatoires

Le socle backend doit progressivement contenir les domaines suivants, chacun isolé dans un module métier clairement identifié :

1. identité et authentification ;
2. utilisateurs, organisations et profils ;
3. KYC, conformité et sanctions ;
4. comptes, portefeuilles et soldes ;
5. grand livre en partie double ;
6. paiements, transferts et demandes de paiement ;
7. cartes physiques, virtuelles et temporaires ;
8. commerçants, points de vente et terminaux ;
9. QR, factures, reçus et partage d’addition ;
10. fidélité, promotions et annuaire ;
11. notifications, support et réclamations ;
12. partenaires bancaires, Mobile Money et réseaux cartes ;
13. services publics, amendes, taxes, bourses et cartes étudiantes ;
14. fraude, risque, limites et blocages ;
15. administration, configuration et drapeaux de fonctionnalités ;
16. audit, reporting, comptabilité et exports.

## Règles de dépendance

- Les applications consomment les contrats partagés mais ne définissent pas les règles financières centrales.
- Le domaine ne dépend d’aucun framework web, mobile ou de persistance.
- Les intégrations externes implémentent des ports du domaine via des adaptateurs remplaçables.
- Les workers réutilisent les cas d’usage du backend et ne dupliquent pas les règles métier.
- Les composants UI ne doivent contenir ni secret, ni logique d’autorisation serveur, ni règle de calcul financier définitive.
- Chaque événement financier doit inclure un identifiant de corrélation, une clé d’idempotence et une version de schéma.

## Environnements et configuration

Les environnements `demo`, `staging` et `production` doivent être séparés. Toute fonctionnalité sensible doit pouvoir être activée ou bloquée selon le pays, le partenaire, le rôle, le canal et l’environnement. Les valeurs réelles sont fournies par un gestionnaire de secrets et ne sont jamais versionnées.

## Vérification de cohérence

Le dépôt plateforme conserve un manifeste lisible par machine dans `docs/module-catalogue.json`. Toute création, suppression ou modification d’un module obligatoire doit mettre à jour simultanément cette cartographie et le manifeste. La CI devra ensuite vérifier que les chemins déclarés existent et que les identifiants restent uniques.

## Critères d’acceptation

- Tous les identifiants de modules sont uniques.
- Chaque application et paquet obligatoire possède un chemin cible explicite.
- Le manifeste du dépôt plateforme représente exactement les mêmes applications et paquets.
- Aucun module métier financier n’est décrit comme dépendant directement d’un fournisseur externe.
- Aucun secret, jeton, mot de passe ou identifiant de production n’apparaît dans les fichiers de cartographie.
