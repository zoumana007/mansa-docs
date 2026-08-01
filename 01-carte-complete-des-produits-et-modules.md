# Carte complète des produits et modules Mansa

Ce document constitue l’inventaire officiel de l’écosystème Mansa. Il sert de liste de contrôle pour éviter qu’une application, un site, un module, une dépendance ou une exigence transverse ne soit oublié pendant la conception ou le développement.

## 1. Applications mobiles

### 1.1 Mansa Client

- inscription, onboarding et authentification ;
- KYC, profil, préférences et sécurité ;
- wallet, comptes, soldes et historique ;
- transferts entre utilisateurs et virements ;
- messagerie Mansa Connect ;
- envoi d’argent et demande d’argent dans une conversation ;
- identifiant unique Mansa, contacts, QR personnel et NFC ;
- cagnottes, groupes financiers et partage d’addition ;
- paiements QR, NFC, cartes et Mobile Money ;
- cartes physiques, virtuelles et jetables ;
- blocage, déblocage, plafonds et mode voyage ;
- Apple Wallet et Google Wallet lorsque les partenariats le permettent ;
- budgets, coffres, objectifs d’épargne et abonnements détectés ;
- reçus numériques, garanties, remboursements et litiges ;
- fidélité, cashback, promotions et avantages ;
- centre familial, compte jeune et contrôle parental ;
- appareils connectés, biométrie et centre de sécurité ;
- documents, services publics, taxes, amendes et cartes étudiantes ;
- bourses, investissements et produits partenaires autorisés ;
- assistant IA Jini ;
- notifications, recherche et accessibilité.

### 1.2 Mansa Commerce

- création d’un commerce et KYB ;
- établissements, employés, rôles et permissions ;
- catalogue, produits, services, variantes, images et prix ;
- taxes, stocks, inventaires, fournisseurs et commandes ;
- ventes, caisse, clients et segmentation ;
- fidélité, promotions, campagnes, coupons et cadeaux ;
- reçus, factures, remboursements, retours, échanges et SAV ;
- codes-barres, QR produits, étiquettes et photos ;
- imprimantes thermiques et modèles de reçus ;
- rendez-vous, planning et disponibilités selon le secteur ;
- mini-site professionnel et profil Annuaire/Hub ;
- statistiques, rapports, abonnements et connexion TPE.

### 1.3 Mansa TPE Android

- activation du terminal et gestion des employés ;
- saisie du montant, catalogue, scan et panier ;
- suppression d’article, modification de quantité, remises et taxes ;
- paiement par carte, NFC, QR, Mobile Money et Tap to Phone ;
- paiement fractionné, partage de facture et pourboires ;
- remboursement, annulation, reçus et réimpression ;
- imprimantes Bluetooth, Wi-Fi, USB ou intégrées ;
- ouverture et clôture de caisse, rapports et rapprochement ;
- mode hors ligne limité et synchronisation ;
- sécurité du terminal ;
- environnements Démo, Recette et Production.

### 1.4 Mansa Admin Lite

- alertes critiques ;
- consultation des incidents et tableaux de bord ;
- validation limitée selon permissions ;
- blocage d’urgence ;
- fraude, support et notifications ;
- approbations mobiles avec traçabilité.

### 1.5 Mansa Annuaire / Hub

- recherche, catégories, carte et géolocalisation ;
- itinéraires, appels, e-mails et WhatsApp ;
- profils professionnels et mini-sites ;
- horaires, photos, produits, services et promotions ;
- favoris, avis et modération ;
- rendez-vous, réservations et disponibilités ;
- établissements à la une ;
- abonnements sectoriels ;
- paiements Mansa ;
- statistiques et monétisation.

## 2. Interfaces web

### 2.1 Site public officiel Mansa

- accueil et présentation de la vision ;
- produits, cartes, paiements et sécurité ;
- partenaires, chiffres clés, utilisateurs et impact ;
- tarifs, actualités, carrière, aide et téléchargements ;
- FAQ, contact et pages légales ;
- CMS, SEO et analytics ;
- animations premium et démonstrations 3D.

### 2.2 Site Mansa Professionnels

- solutions commerçants et entreprises ;
- TPE, Tap to Phone, paiements et abonnements ;
- catalogue, stocks, fidélité et promotions ;
- Annuaire/Hub et mini-sites ;
- tarifs, démonstrations, demande de devis et onboarding ;
- espace partenaire, documentation commerciale, formulaires et leads ;
- CMS et analytics.

### 2.3 Portail Admin Web

- dashboard global ;
- utilisateurs, commerçants, entreprises, agents et employés ;
- rôles, permissions, RBAC et ABAC ;
- KYC, KYB, wallets, comptes et cartes ;
- transactions, transferts, paiements et TPE ;
- Mobile Money, remboursements, litiges et fraude ;
- support, notifications, abonnements et commissions ;
- taxes, catalogues, promotions et CMS des deux sites ;
- gestion de l’Annuaire/Hub ;
- partenaires, banques, pays, devises et langues ;
- module État, amendes, bourses et cartes étudiantes ;
- investissements et IA Jini ;
- analytics, audit, logs, monitoring et incidents ;
- feature flags, environnements, exports et conformité ;
- double validation pour les opérations critiques.

## 3. Backend et services communs

- API Gateway ;
- authentification, sessions, RBAC, ABAC et audit ;
- utilisateurs, identité, KYC et KYB ;
- wallets, grand livre, transactions et transferts ;
- paiements, cartes, NFC, QR et Mobile Money ;
- TPE, commerçants, catalogue, stocks, promotions et fidélité ;
- reçus, factures, retours, remboursements et litiges ;
- messagerie, notifications, support et fraude ;
- analytics, IA, services publics et investissements ;
- documents, recherche, géolocalisation et rendez-vous ;
- abonnements, commissions, taxes, webhooks et partenaires ;
- fichiers, tâches asynchrones, idempotence, corrélation, cache et files de messages.

## 4. Packages partagés

- contrats API ;
- types ;
- SDK ;
- utilitaires ;
- design system et composants UI ;
- thèmes et internationalisation ;
- permissions et sécurité ;
- paiements et notifications ;
- analytics, validation, erreurs, configuration et feature flags.

## 5. Infrastructure

- monorepo PNPM et Turborepo ;
- PostgreSQL, Prisma et Redis lorsque nécessaire ;
- Docker et environnements de développement ;
- CI, formatage, lint, tests et build ;
- déploiement, monitoring, logs et alertes ;
- sauvegardes, migrations et observabilité ;
- gestion externe des secrets ;
- documentation d’exploitation.

## 6. Modules dépendants de partenaires ou d’autorisations

- Visa et Mastercard ;
- émission et acquisition de cartes ;
- Mobile Money ;
- Tap to Phone ;
- Apple Wallet et Google Wallet ;
- services publics, taxes et amendes ;
- bourses et cartes étudiantes ;
- investissements et levées de fonds ;
- SMS, WhatsApp et e-mail ;
- logistique, livraison et points relais.

## 7. Règle anti-oubli

Chaque module devra disposer, avant développement complet, des éléments suivants :

- objectifs ;
- utilisateurs concernés ;
- écrans et parcours ;
- règles métier ;
- contrats API ;
- modèles de données ;
- rôles et permissions ;
- administration ;
- notifications ;
- sécurité ;
- cas d’erreur ;
- critères d’acceptation ;
- tests ;
- dépendances ;
- statut.

Aucun module ne doit être considéré comme complet tant que ces éléments ne sont pas documentés ou explicitement marqués comme non applicables.