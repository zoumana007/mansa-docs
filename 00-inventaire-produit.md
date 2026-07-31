# Inventaire produit complet Mansa

Ce document est la source de vérité fonctionnelle du programme Mansa. Toute application, tout site, tout service backend et tout module transversal doit être relié à une ligne de cet inventaire. Une fonctionnalité non répertoriée ici doit être ajoutée avant développement.

## 1. Produits utilisateurs et opérateurs

### 1.1 Application mobile Client

Périmètre minimal : inscription, authentification forte, KYC, portefeuille, solde, historique, virements, paiements QR et NFC lorsque compatibles, cartes physiques et virtuelles, carte jetable/temporaire, Mobile Money, RIB/IBAN selon pays, reçus, factures, budgets, coffres et objectifs, détection d’abonnements, cashback, fidélité, promotions, gestion des bénéficiaires, notifications, support, assistant Jini, paramètres de sécurité, appareils de confiance et fermeture de session distante.

### 1.2 Application mobile Commerçant

Périmètre minimal : onboarding KYB, profil établissement, encaissement, QR statique/dynamique, liens de paiement, catalogue, produits et services, factures, remboursements, gestion des employés, rôles, caisses, rapprochement, rapports, promotions, fidélité, mini-site, commandes, mode hors ligne contrôlé, synchronisation, support, litiges et export comptable.

### 1.3 Application TPE Android

Périmètre minimal : paiement carte EMV, NFC, QR et wallet selon certifications et partenaires, saisie du montant, pourboire configurable, partage ou division d’addition, préautorisation si activée, remboursement, annulation, paiement hors ligne sous limites de risque, impression ou envoi de reçu, clôture de caisse, diagnostics, gestion distante, mises à jour signées, association au commerçant et mode Démo/Recette/Production.

### 1.4 Application Admin Lite mobile

Périmètre minimal : alertes critiques, validation à double contrôle, supervision rapide, incidents, fraude, KYC/KYB, support prioritaire, blocage contrôlé, consultation d’audit, statut partenaires et actions d’urgence limitées par permissions.

### 1.5 Application mobile Annuaire/Hub

Produit séparé de l’application Client et de l’application Commerçant. Périmètre minimal : recherche par catégorie, texte et proximité, géolocalisation consentie, carte, filtres, profils professionnels, mini-sites automatiques, horaires, contacts, itinéraires, offres, promotions, établissements à la une, favoris, avis et modération, réservation ou prise de contact selon le secteur, paiements Mansa, comptes particuliers et professionnels, abonnements sectoriels, règles de visibilité, monétisation, statistiques et administration complète.

## 2. Interfaces web

### 2.1 Site public officiel Mansa

Pages et fonctions minimales : accueil, vision, produits, cartes, paiements, sécurité, particuliers, impact, chiffres clés, utilisateurs, partenaires, tarifs, actualités, carrières, aide, téléchargement, contact, presse, pages légales, consentement cookies, SEO, analytics, CMS et internationalisation. Les chiffres publiés doivent provenir de sources administrées et validées.

### 2.2 Site professionnel / commerçants / partenaires

Pages et fonctions minimales : offres commerçants, TPE, paiements, catalogue, mini-sites, fidélité, intégrations, tarification, démonstrations, onboarding, demande de devis, génération de leads, formulaires, documentation commerciale, espace partenaire, cas clients, support, CMS, analytics et intégration au backend.

### 2.3 Portail Admin Web

Périmètre minimal : dashboard global, utilisateurs, commerçants, TPE, transactions, cartes, wallets, KYC/KYB, rôles et permissions RBAC/ABAC, audit, fraude, litiges, support, notifications, CMS des deux sites, administration Annuaire/Hub, configuration fonctionnelle, tarification, commissions, pays, devises, langues, banques, Mobile Money, partenaires, module État, bourses, taxes, amendes, cartes étudiantes, IA/Jini, analytics, monitoring, incidents, feature flags, environnements Démo/Recette/Production, exports, rapports, conformité et journaux techniques. Les opérations critiques exigent une permission explicite, une justification, une trace d’audit et, selon criticité, une double validation.

## 3. Backend et services

### 3.1 API Gateway et identité

API versionnées, authentification, autorisation, rate limiting, idempotence, validation, corrélation, journalisation, gestion de session, appareils, MFA, consentements, webhooks signés et séparation des environnements.

### 3.2 Domaine financier

Grand livre en partie double, wallets, comptes, réservations, autorisations, captures, annulations, remboursements, frais, commissions, rapprochement, settlement, limites, taux, devises, cartes, Mobile Money, banques, QR, NFC, TPE, reçus, factures et litiges. Aucun solde ne doit être modifié sans écriture comptable équilibrée.

### 3.3 Conformité et risque

KYC, KYB, sanctions, PEP, fraude, scoring, limites, règles, revues manuelles, dossiers, preuves, conservation, audit, consentements et centre de conformité. Les règles juridiques finales restent à valider par pays et partenaire.

### 3.4 Services IA

Assistant Jini, support augmenté, classification, recherche, résumé, recommandation et aide à la détection de fraude. Toute décision réglementaire ou financière sensible doit rester explicable, supervisable et validée selon les règles métier. Les modèles ne reçoivent pas de secrets ni de données personnelles non nécessaires.

### 3.5 Module État

Amendes routières, taxes, paiements publics, bourses, cartes étudiantes et services administratifs. Exigences obligatoires : identité agent, terminal enregistré, référentiel d’infractions, preuve, reçu, paiement traçable, séparation des rôles, impossibilité de modifier silencieusement une opération, journal d’audit, rapprochement et mécanismes anticorruption.

## 4. Modules transversaux

- Notifications multicanales et préférences.
- Support, tickets, chat et SLA.
- CMS et publication avec validation.
- Analytics, BI, exports et rapports.
- Observabilité, logs, métriques et traces.
- Feature flags et configuration distante.
- Internationalisation, multi-pays, multi-devises et multi-langues.
- Design system partagé et accessibilité.
- Sécurité, secrets externes, sauvegardes, reprise et continuité.
- CI/CD, qualité, tests, scans et déploiements contrôlés.

## 5. Exigences de spécification par produit

Chaque produit doit disposer des sections suivantes :

1. objectifs et utilisateurs ;
2. rôles et permissions ;
3. parcours et écrans ;
4. états vides, erreurs et cas limites ;
5. règles métier ;
6. modèle de données ;
7. contrats API et événements ;
8. administration et configuration ;
9. sécurité, conformité et audit ;
10. observabilité ;
11. tests et critères d’acceptation ;
12. stratégie de migration et déploiement.

## 6. Éléments DeepSeek non récupérés intégralement

À compléter depuis l’export DeepSeek : texte exact des prompts historiques, variantes rejetées, extraits de code éventuellement utiles et décisions non présentes dans l’historique accessible. Ces éléments ne doivent pas être inventés. Ils seront intégrés dans un dossier d’archives puis consolidés dans les spécifications actives.