# Inventaire produit consolidé Mansa

Ce document est la liste de contrôle obligatoire du périmètre. Aucun agent IA ne doit supprimer, fusionner ou ignorer un produit sous prétexte qu’il n’est pas encore implémenté.

## Applications mobiles

1. `mobile-client` — portefeuille, cartes, paiements, virements, budgets, coffres, abonnements, fidélité, QR, NFC, Mobile Money, support et Jini.
2. `mobile-merchant` — encaissement, catalogue, facturation, employés, caisse, fidélité, promotions, mini-site, reporting et gestion TPE.
3. `mobile-admin-lite` — supervision mobile, alertes, approbations, incidents, support et actions urgentes sous permissions fines.
4. `mobile-directory` — Annuaire/Hub autonome avec recherche géolocalisée, catégories, profils professionnels, mini-sites, favoris, avis, offres, mises en avant, contacts, réservations sectorielles, paiements Mansa, monétisation et statistiques.
5. `tpe-android` — application Android pour terminaux compatibles, paiement carte/NFC/QR, partage d’addition, remboursement, mode hors ligne contrôlé, reçus et intégration matérielle isolée.

## Interfaces web

1. `public-web` — site officiel public Mansa.
2. `business-web` — site dédié aux commerçants, partenaires, TPE et services professionnels.
3. `admin-web` — portail d’administration complet, distinct des deux sites publics.

## Backend et services

- API Gateway NestJS.
- Modules métier : identité, KYC/KYB, utilisateurs, organisations, wallets, ledger, paiements, transferts, cartes, TPE, commerçants, annuaire, fidélité, promotions, factures, abonnements, support, notifications, litiges, fraude, conformité, module État, bourses, taxes, amendes, cartes étudiantes, reporting et configuration.
- Services IA : Jini, support assisté, détection d’anomalies, fraude, recommandations et modération sous contrôle humain.
- Workers : files de messages, webhooks, notifications, rapprochements, exports, traitements planifiés et tâches longues.

## Plateforme partagée

- Contrats, DTO et événements versionnés.
- Design system et primitives d’animation.
- Sécurité, permissions RBAC/ABAC et audit.
- Observabilité, logs, traces, métriques et corrélation.
- Infrastructure Démo, Recette et Production.
- CI/CD de validation sans secret.

## Règle de traçabilité

Chaque fonctionnalité doit être reliée à :

- une application ou un service propriétaire ;
- des rôles autorisés ;
- un workflow nominal et des erreurs ;
- des entités de données ;
- des endpoints ou événements ;
- des règles de sécurité et d’audit ;
- des critères d’acceptation ;
- des tests.

## Éléments DeepSeek

Tout contenu exact non encore récupéré depuis l’export DeepSeek doit être placé dans une section `À compléter depuis l’export DeepSeek`. Il est interdit d’inventer un ancien prompt puis de le présenter comme provenant de DeepSeek.