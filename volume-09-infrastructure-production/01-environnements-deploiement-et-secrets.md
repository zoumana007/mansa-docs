# Volume 9 — Environnements, déploiement et secrets

## 1. Objectif

Ce document définit les règles minimales permettant de faire évoluer Mansa du développement local vers des environnements hébergés sans mélanger données, identités, clés ni partenaires.

## 2. Environnements

Mansa utilise au minimum quatre contextes séparés :

- `local` : poste développeur, données fictives et services simulés ;
- `demo` : démonstrations commerciales, jeux de données réinitialisables et aucune valeur réelle ;
- `staging` : recette technique et métier, configuration proche de la production mais partenaires de test ;
- `production` : services réels, accès restreints, supervision et procédures d’urgence actives.

Chaque environnement possède ses propres bases, files, stockage objet, clés de signature, domaines, applications OAuth, comptes partenaires et journaux.

## 3. Chaîne de livraison

1. Une modification est développée sur une branche courte.
2. La CI exécute formatage, lint, vérification TypeScript, tests, validation Prisma et build.
3. La revue contrôle le code, les migrations, les contrats publics et les risques de sécurité.
4. L’artefact est construit une seule fois et identifié par le SHA du commit.
5. Le même artefact est promu vers `demo`, puis `staging`, puis `production`.
6. La mise en production exige une approbation explicite et un plan de retour arrière.
7. Le déploiement et son résultat sont consignés dans l’audit d’exploitation.

## 4. Configuration

La configuration non sensible peut être versionnée : noms de modules, valeurs par défaut prudentes, limites de démonstration et options désactivées.

Les éléments suivants ne doivent jamais être versionnés :

- mots de passe et chaînes de connexion réelles ;
- clés API, secrets OAuth et jetons ;
- clés privées de signature ou de chiffrement ;
- certificats clients ;
- identifiants de comptes partenaires ;
- données personnelles ou exports de production.

Les secrets sont injectés au démarrage par le gestionnaire de secrets de la plateforme. Toute valeur sensible doit pouvoir être renouvelée sans reconstruire le code.

## 5. Migrations de données

- Toute migration Prisma est relue avant fusion.
- Les migrations destructives sont séparées en plusieurs déploiements compatibles.
- Le code doit tolérer temporairement l’ancien et le nouveau schéma.
- Une sauvegarde vérifiée est réalisée avant une migration risquée.
- Les migrations de production ne sont jamais lancées automatiquement depuis un poste local.
- Le retour arrière applicatif ne doit pas aggraver une migration déjà appliquée.

## 6. Stratégie de déploiement

Le déploiement progressif est privilégié : rolling update, canary ou blue/green selon l’hébergeur. Les changements à risque sont protégés par des feature flags configurables dans l’administration.

Les critères d’arrêt automatique incluent : hausse du taux d’erreur, latence excessive, échec des vérifications de santé, dérive des transactions ou échec de connexion aux dépendances critiques.

## 7. Séparation des responsabilités

- Les développeurs peuvent administrer `local` et contribuer à `demo`.
- L’accès à `staging` est nominatif et limité aux besoins de recette.
- La production applique le moindre privilège et l’authentification forte.
- Une personne ne doit pas pouvoir développer, approuver et déployer seule un changement financier critique.
- Les accès d’urgence sont temporaires, justifiés et audités.

## 8. Vérifications avant production

- Variables obligatoires présentes et validées au démarrage.
- Aucun secret de démonstration utilisé en production.
- Sauvegardes et restauration testées.
- Tableaux de bord, alertes et astreinte configurés.
- Migrations validées sur une copie représentative.
- Intégrations partenaires testées dans leur environnement homologué.
- Runbooks d’incident et de retour arrière disponibles.
- Tests de sécurité et revue réglementaire terminés.

## 9. Éléments manuels obligatoires

Le code ne peut pas remplacer les contrats avec banques, opérateurs, processeurs de cartes et administrations, ni les agréments, audits indépendants, cérémonies de clés et validations de conformité nécessaires avant l’utilisation d’argent réel.
