# Inventaire fonctionnel complet — Mansa

Ce document est la liste de contrôle centrale du projet. Une fonction n’est considérée comme couverte que lorsqu’elle existe dans la documentation, les contrats, le code, les tests et les écrans concernés.

## 1. Produits et surfaces

### Applications mobiles
- Application Client : inscription, KYC, portefeuille, cartes, paiements, transferts, QR, NFC, Mobile Money, budgets, coffres, abonnements, reçus, factures, fidélité, cashback, promotions, support et assistant Jini.
- Application Commerçant : encaissement, catalogue, stock, caisse, factures, remboursements, employés, rôles, fidélité, promotions, mini-site, analytics, rapprochement et support.
- Application Admin Lite : alertes, validations urgentes, supervision, incidents, utilisateurs, agents et opérations sensibles.
- Application Annuaire / Hub : découverte géolocalisée, catégories, recherche, abonnements sectoriels, profils à la une, mini-sites et prise de contact.
- Application TPE Android : paiement carte, NFC, QR, Mobile Money, saisie manuelle autorisée, annulation, remboursement, pourboire, partage d’addition, reçus, mode hors ligne contrôlé et synchronisation.

### Sites web
- Site public Mansa : présentation des services, sécurité, partenaires, tarification, statistiques configurables, actualités, assistance, téléchargement des applications, pages légales et espace presse.
- Portail Admin Web : gouvernance complète, configuration, utilisateurs, commerçants, agents, transactions, cartes, risques, support, contenu, partenaires, commissions, limites, audit, analytics et déploiements.
- Portail Commerçant Web : gestion avancée du commerce, équipes, catalogue, ventes, facturation, paiements, règlements, litiges et rapports.
- Portail Développeurs : documentation API, clés, webhooks, SDK, environnement sandbox, journaux et exemples.
- Portail Partenaires / Institutions : banques, opérateurs, établissements, organismes publics et programmes dédiés.
- Mini-sites commerçants générés : vitrine, catalogue, horaires, localisation, promotions, paiement et contact.

### Services backend
- API Gateway et authentification.
- Identité, KYC/KYB, consentements et conformité.
- Comptes, portefeuille, grand livre en partie double et rapprochement.
- Paiements, transferts, cartes, QR, NFC et Mobile Money.
- Commerçants, TPE, catalogue, commandes, facturation et règlements.
- Fidélité, cashback, promotions, budgets, coffres, tontines et abonnements.
- Notifications, support, litiges, remboursements et preuves.
- IA Jini, fraude, scoring, recommandations et automatisations internes.
- Module État et services publics.
- Annuaire professionnel et recherche.
- Analytics, reporting, exports, comptabilité et BI.
- Administration, configuration, feature flags et audit immuable.

## 2. Paiements et argent
- Portefeuilles multi-devises.
- Rechargement et retrait via banque, Mobile Money et partenaires autorisés.
- Transfert interne instantané par téléphone, identifiant, QR ou lien.
- Virement bancaire et bénéficiaires.
- Paiement marchand en ligne, en magasin et à distance.
- Paiement NFC téléphone-à-téléphone lorsque matériel et certification le permettent.
- QR statique et dynamique.
- Cartes physiques, virtuelles, temporaires et à usage unique selon partenaire émetteur.
- Gel, dégel, plafonds, géoblocage, catégories, paiements en ligne et sans contact.
- Paiements récurrents et détection d’abonnements.
- Demande d’argent, partage d’addition et collecte de groupe.
- Annulation, remboursement total ou partiel, rétrofacturation et litige.
- Reçus, factures, preuves horodatées et export PDF.
- Frais, commissions et partage de revenus entièrement configurables et versionnés.

## 3. Client
- Onboarding progressif avec récupération de session.
- KYC par niveaux et limites associées.
- Accueil personnalisable avec solde, actions rapides et activité.
- Historique filtrable et recherche.
- Contacts, favoris et bénéficiaires.
- Budgets par catégorie et alertes.
- Coffres personnels et partagés avec objectifs.
- Tontines numériques avec règles, échéances, participants et traçabilité.
- Carte de fidélité numérique intégrée aux cartes et avantages.
- Cashback, coupons, points et offres géolocalisées.
- Gestion des abonnements et dépenses récurrentes.
- Centre de sécurité, appareils, sessions et autorisations.
- Support multicanal et assistant Jini avec transfert à un humain.

## 4. Commerçant et TPE
- Création du commerce, KYB, établissements et points de vente.
- Catalogue, variantes, taxes, remises, stock et codes-barres.
- Vente rapide et panier détaillé.
- Addition commune, séparation égale, par article ou par montant.
- Pourboires configurables.
- Encaissement multi-moyens et paiement partiel.
- Mode restaurant, commerce, service, transport et autres secteurs configurables.
- Employés, caissiers, managers, permissions et horaires.
- Ouverture/fermeture de caisse et écarts.
- Remboursements soumis à permissions et justification.
- Règlements commerçants, commissions, rapprochement et exports.
- Fonctionnement dégradé hors ligne avec plafonds, liste de risques et synchronisation idempotente.
- Gestion distante des terminaux, mises à jour et révocation.

## 5. Administration et gouvernance
- Super Admin avec séparation des responsabilités.
- Administrateurs par domaine, pays, partenaire, établissement et rôle.
- Matrice RBAC/ABAC configurable.
- Double validation pour actions critiques.
- Journal d’audit append-only : acteur, date, appareil, IP, ancienne valeur, nouvelle valeur et justification.
- Configuration centralisée des limites, frais, commissions, contenus, pays, devises, langues et fonctionnalités.
- Feature flags avec ciblage et rollback.
- Gestion des incidents, maintenance et messages in-app.
- Vue à 360° client, commerçant, agent, partenaire et transaction.
- Outils de blocage réellement appliqués dans tous les services et caches.
- Exports contrôlés, filigranés et tracés.

## 6. Module État et institutions
- Paiement d’amendes sur place et à distance.
- Identification forte de l’agent, terminal enregistré et habilitation active.
- Catalogue officiel des infractions, montants versionnés et impossibilité de modifier librement le prix par l’agent.
- Procès-verbal numérique, référence unique, reçu immédiat et preuve vérifiable.
- Paiement de taxes, redevances, frais scolaires et services publics.
- Bourses : listes de bénéficiaires, validation, versement, rejet, reprise et audit.
- Cartes étudiantes physiques ou numériques et statut d’établissement.
- Portails institutionnels cloisonnés.
- Réconciliation avec les systèmes publics et partenaires bancaires.
- Mécanismes anticorruption : interdiction d’effacement, supervision, anomalies, géolocalisation autorisée, séparation encaissement/annulation et alertes.

## 7. Intelligence artificielle
- Jini : aide contextuelle, navigation, explication des transactions et support.
- Aucune exécution financière irréversible sans consentement explicite et contrôle serveur.
- Détection de fraude et d’anomalies avec règles explicables.
- Résumés, catégorisation et recommandations budgétaires.
- Assistance commerçant : analyse des ventes, stock et actions suggérées.
- Assistance agents support et administrateurs sans exposition excessive de données.
- Registre des modèles, versions, prompts, jeux de tests, décisions et mécanismes de désactivation.

## 8. Design et expérience
- Design premium, moderne et cohérent avec une fintech avancée.
- Tous les textes, images de campagne, statistiques publiques et blocs marketing modifiables depuis l’administration.
- Animations fluides et sobres avec respect de la préférence de réduction des mouvements.
- Écrans d’avant connexion avec visuels animés ou effets de profondeur optimisés.
- États chargement, vide, erreur, hors ligne, succès, attente et révision manuelle pour chaque parcours.
- Accessibilité, contraste, tailles tactiles et navigation clavier sur le web.
- Design tokens partagés et composants documentés.

## 9. Technique, sécurité et exploitation
- Monorepo pnpm, TypeScript strict et contrats partagés.
- NestJS, PostgreSQL et Prisma pour le socle backend.
- Next.js pour les portails web.
- React Native pour les applications mobiles ; Android natif ou module natif pour les fonctions TPE certifiées.
- Authentification forte, MFA, biométrie locale, gestion des sessions et appareils.
- Chiffrement en transit et au repos, secrets externes, rotation et moindre privilège.
- Idempotence, outbox, événements versionnés et reprise sur erreur.
- Observabilité : logs structurés, métriques, traces, alertes et corrélation.
- Sauvegardes, restauration testée, PRA/PCA et haute disponibilité selon criticité.
- Environnements Démo, Recette et Production strictement séparés.
- CI : format, lint, typecheck, tests, build, migrations et contrôles de sécurité sans déploiement automatique non autorisé.

## 10. Définition de « complet »
Pour chaque élément :
1. exigence documentée ;
2. règles métier et permissions définies ;
3. contrat API ou événement défini ;
4. modèle de données défini ;
5. interface et états UX définis ;
6. implémentation ;
7. tests unitaires, intégration et parcours critique ;
8. observabilité et audit ;
9. critères d’acceptation validés ;
10. documentation d’exploitation.
