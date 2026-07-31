# Volume 01 — Vision, périmètre et principes du socle

## 1. Objet

Mansa est une plateforme fintech modulaire destinée au Mali puis à l’Afrique de l’Ouest. Elle réunit dans un même écosystème les particuliers, commerçants, agents, entreprises, banques, opérateurs Mobile Money et administrations publiques.

## 2. Objectifs du socle

- Permettre les paiements, transferts, encaissements et décaissements multicanaux.
- Fournir des applications distinctes mais interconnectées : Client, Commerçant, TPE, Admin Lite, Annuaire, portail web et site public.
- Offrir une administration centrale capable de configurer produits, commissions, limites, rôles, partenaires et règles par pays.
- Assurer une traçabilité complète des opérations et des actions administratives.
- Supporter une montée en charge progressive jusqu’à plusieurs millions d’utilisateurs.
- Intégrer les services publics : amendes, taxes, bourses, scolarité, cartes étudiantes et paiements administratifs.

## 3. Acteurs principaux

- Client particulier.
- Commerçant et employé de commerce.
- Agent terrain et agent public.
- Administrateur métier.
- Super administrateur.
- Banque partenaire.
- Opérateur Mobile Money.
- Réseau de cartes et processeur de paiement.
- Établissement public, université ou collectivité.
- Équipe support, conformité, risque et finance.

## 4. Périmètre fonctionnel initial

Le premier périmètre couvre : authentification, KYC, comptes, portefeuilles, paiements, transferts, QR, cartes, terminaux TPE, gestion commerçant, notifications, support, audit, administration, intégrations partenaires et reporting.

Les fonctionnalités avancées sont activées progressivement par configuration et drapeaux de fonctionnalités.

## 5. Principes non négociables

1. Aucun montant financier n’est représenté en nombre flottant.
2. Toute transaction possède un identifiant idempotent et un journal d’événements.
3. Toute action sensible est authentifiée, autorisée et auditée.
4. Les environnements Démo, Recette et Production sont strictement séparés.
5. Les secrets sont injectés par le gestionnaire de secrets de l’environnement.
6. Les règles métier sont configurables sans modification directe de la base.
7. Toute intégration externe est isolée derrière un adaptateur.
8. Le système doit pouvoir bloquer immédiatement une fonction, un compte, un partenaire ou un pays.
9. Le grand livre financier suit la partie double et reste la source de vérité comptable.
10. Les contrats publics sont versionnés et compatibles avec les clients pris en charge.

## 6. Hors périmètre du premier socle

- Obtention automatique d’un agrément financier.
- Compensation bancaire réelle sans contrat partenaire.
- Émission de cartes en production sans émetteur et processeur agréés.
- Déploiement public avant audits de sécurité, conformité et reprise après sinistre.

## 7. Critères de réussite

- Une base de code reproductible et testable.
- Une architecture documentée et cohérente avec les applications.
- Une API versionnée et documentée.
- Des contrôles de sécurité automatisés.
- Une traçabilité complète des transactions et changements administratifs.
- Une capacité à ajouter un pays ou un partenaire sans réécrire le cœur métier.
- Une commande de validation unique couvrant formatage, lint, types, tests et build.

## 8. Correspondance avec le dépôt plateforme

Cette spécification correspond au monorepo `zoumana007/mansa-platform` et notamment aux chemins suivants :

- `apps/api-gateway` pour l’API et l’orchestration.
- `apps/workers` pour les traitements asynchrones.
- `apps/ai-services` pour les services IA contrôlés.
- `packages/contracts` pour les contrats partagés.
- `packages/domain` pour les primitives métier.
- `packages/security` pour les règles d’autorisation.
- `packages/observability` pour la journalisation, les métriques et les traces.
- `infra` pour les environnements et le déploiement.

Toute évolution de ces chemins ou responsabilités doit être reportée dans les deux dépôts au cours du même lot de travail.