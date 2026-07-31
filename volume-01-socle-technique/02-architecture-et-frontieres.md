# Architecture et frontières du socle

## 1. Style d’architecture

Mansa adopte un monorepo modulaire et une architecture hexagonale. Le domaine financier reste indépendant de NestJS, Next.js, React Native, Android, Prisma et des fournisseurs externes.

Les dépendances sont orientées vers le cœur :

1. domaine et règles métier ;
2. cas d’usage applicatifs ;
3. ports de persistance et d’intégration ;
4. adaptateurs techniques ;
5. interfaces HTTP, événements, web et mobiles.

## 2. Contextes métier

- Identité et accès : comptes, sessions, MFA, rôles et permissions.
- Conformité : KYC, sanctions, limites, risque et dossiers de contrôle.
- Wallet et ledger : portefeuilles, soldes disponibles, écritures et rapprochement.
- Paiements : transferts, QR, cartes, Mobile Money, TPE et remboursements.
- Commerce : établissements, employés, catalogues, encaissements et fidélité.
- Administration : configuration, validation à quatre yeux, audit et support.
- Services publics : amendes, taxes, scolarité, bourses et cartes étudiantes.
- Notifications : push, SMS, e-mail et messages applicatifs.
- Données et IA : reporting, fraude, Jini, recommandations et assistance.

Chaque contexte expose des contrats explicites. Aucun module ne lit directement les tables internes d’un autre contexte.

## 3. Responsabilités des applications

- `apps/api-gateway` expose les API synchrones, applique authentification et autorisation, puis appelle les cas d’usage.
- `apps/workers` traite les événements, reprises, notifications, rapprochements et tâches longues.
- `apps/ai-services` isole les fonctions IA et ne peut modifier un solde directement.
- Les applications mobiles et web consomment les contrats versionnés ; elles ne contiennent pas les règles financières faisant autorité.

## 4. Responsabilités des paquets

- `packages/domain` : entités, objets-valeur, invariants, ports et services applicatifs sans framework.
- `packages/contracts` : DTO, schémas de validation, événements et erreurs publiques versionnés.
- `packages/security` : politiques RBAC/ABAC, permissions, décisions et primitives de sécurité.
- `packages/observability` : corrélation, journalisation structurée, métriques et traces.
- `packages/config` : configurations partagées de compilation, qualité et outillage.
- `packages/ui` : composants visuels sans règle métier financière.

## 5. Règles de dépendance

- Le domaine ne dépend d’aucune application ni infrastructure.
- Les contrats ne dépendent pas des interfaces utilisateur.
- Un adaptateur externe implémente un port défini par le domaine ou l’application.
- Les appels entre contextes passent par un service publié ou un événement versionné.
- Une transaction financière ne dépend jamais de la disponibilité d’un service IA.
- Les opérations longues sont asynchrones et observables.

## 6. Transactions et événements

Une modification financière atomique écrit le ledger et l’événement sortant dans la même transaction locale via un mécanisme d’outbox. Les consommateurs sont idempotents et conservent la clé de déduplication. Les événements publiés sont immuables ; une correction produit un nouvel événement.

## 7. Sécurité des frontières

Chaque entrée externe doit appliquer : validation du schéma, authentification, autorisation, limitation de débit, corrélation, idempotence lorsque nécessaire et journal d’audit pour les actions sensibles.

Les secrets, clés privées, jetons partenaires et données KYC réelles ne sont jamais stockés dans Git.

## 8. Critères d’acceptation

- Les imports interdits entre couches sont détectés par lint ou tests d’architecture.
- Le domaine peut être testé sans démarrer NestJS ni une base de données.
- Chaque intégration possède une interface, un adaptateur et un faux de test.
- Les contrats publics sont versionnés et accompagnés d’un test de compatibilité.
- Le chemin de chaque module documenté existe ou est marqué explicitement comme cible à créer.
