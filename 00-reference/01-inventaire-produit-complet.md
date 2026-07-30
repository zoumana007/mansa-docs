# Inventaire produit complet — Mansa

Ce document est la référence obligatoire utilisée par Codex, VS Code IA et toute équipe de développement. Une fonctionnalité n’est considérée couverte que si elle possède une documentation fonctionnelle, des règles métier, des contrats techniques, des critères d’acceptation et des tests.

## 1. Produits utilisateurs

### 1.1 Application Client
Wallet multi-devises, inscription et KYC, authentification forte, virements, QR, NFC, paiements, cartes physiques et virtuelles, carte temporaire ou jetable, coffres et objectifs, budgets, abonnements détectés, factures et reçus, partage d’addition, cashback, fidélité, promotions, historique, litiges, support, notifications, profil, paramètres de sécurité, appareils autorisés, intégration Mobile Money, banque, Visa/Mastercard et Wallets mobiles.

### 1.2 Application Commerçant
Onboarding entreprise, KYC/KYB, encaissement, QR statique et dynamique, liens de paiement, catalogue, commandes, factures, reçus, remboursements, pourboires, partage d’addition, fidélité, promotions, gestion des employés, rôles et caisses, mini-site marchand, stocks simples, analytics, rapprochement, reversements, support et paramètres.

### 1.3 Application TPE Android
Encaissement carte, NFC, QR et wallet Mansa, saisie du montant, panier, pourboire, partage ou séparation de note, paiement partiel, annulation, remboursement autorisé, reçus, mode hors ligne contrôlé, synchronisation, gestion de caisse, employés, journal technique, mises à jour à distance et configuration par l’administration. Cible matérielle initiale : terminaux Android de type PAX A920 Pro, sans dépendance métier à un seul constructeur.

### 1.4 Application Admin Lite mobile
Alertes, indicateurs essentiels, approbations limitées, incidents, tickets prioritaires, consultation d’audits, suspension contrôlée, validation à quatre yeux selon politique et actions d’urgence traçables. Elle ne remplace pas le portail Admin Web.

### 1.5 Application Annuaire / Hub
Produit mobile séparé. Recherche géolocalisée, catégories et secteurs, profils professionnels, mini-sites automatiques, offres, promotions, établissements à la une, favoris, avis et modération, itinéraires, contacts, réservation ou demande de devis selon secteur, paiement Mansa, comptes utilisateurs et professionnels, abonnements sectoriels, règles de visibilité, monétisation, statistiques et administration complète.

## 2. Produits web

### 2.1 Portail Admin Web
Super administration, utilisateurs, commerçants, agents, partenaires, État, KYC/KYB, transactions, cartes, risques, fraude, litiges, support, contenu, promotions, annuaire, CMS des deux sites, tarification, commissions, pays, devises, langues, feature flags, journaux d’audit, rapports, rapprochements, intégrations, clés techniques, webhooks, appareils, TPE, mises à jour, incidents et gouvernance.

### 2.2 Site public officiel Mansa
Site institutionnel grand public : vision, produits, fonctionnalités, sécurité, tarifs, chiffres clés, impact, revenus publiables, nombre d’utilisateurs, partenaires, actualités, carrière, centre d’aide, téléchargements, pages légales, statut des services, presse, SEO, analytics, consentement, CMS administrable et animations premium.

### 2.3 Site Commerçants & Partenaires
Second site distinct : acquisition commerçants, offres TPE, paiements, mini-sites, catalogue, fidélité, tarifs, simulateurs, démonstrations, documentation commerciale, onboarding, génération de prospects, formulaires, espace partenaire, cas clients, intégrations, CMS, analytics et connexion au backend.

## 3. Plateforme et services

- API Gateway NestJS.
- Modules métier financiers.
- Services IA : assistant Jini, support assisté, fraude, classification, recommandations et résumé contrôlé.
- PostgreSQL avec Prisma.
- Stockage d’objets et pièces justificatives.
- Files de messages et événements métier.
- Notifications push, SMS, email et in-app via adaptateurs.
- Connecteurs banque, Mobile Money, cartes, KYC, sanctions, change, comptabilité et services publics.
- Packages partagés : contrats, domaine, configuration, observabilité, sécurité et composants UI.
- Infrastructure, CI/CD, observabilité, sauvegardes, reprise, haute disponibilité et gestion des secrets.

## 4. Module État

Amendes routières sur place, identification forte des agents, infractions et barèmes administrables, preuve, reçu immédiat, paiement numérique, contestation, traçabilité anti-corruption, taxes, frais administratifs, scolarité, cartes étudiantes, bourses, décaissements, rapprochement, rôles institutionnels, séparation des pouvoirs et audit immuable.

## 5. Exigences transversales

- Multi-pays, multi-entité, multi-devise et multi-langue.
- Modes Démo, Recette et Production strictement séparés.
- RBAC et ABAC selon contexte.
- Double validation pour opérations sensibles.
- Idempotence, ledger en partie double et montants en unités mineures.
- Paramétrage administratif avec historique des versions et possibilité de retour arrière.
- Protection des données, minimisation, consentement, rétention et export.
- Accessibilité, faible débit, résilience réseau et appareils d’entrée de gamme.
- Toutes les opérations critiques génèrent une piste d’audit.

## 6. Sources manquantes

Les contenus exacts qui ne sont pas présents dans l’historique accessible doivent être marqués : **À compléter depuis l’export DeepSeek**. Aucun texte absent ne doit être inventé ou présenté comme un prompt original.