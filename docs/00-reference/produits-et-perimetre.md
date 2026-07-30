# Référentiel des produits Mansa

Ce document est la source de vérité fonctionnelle du programme Mansa. Aucun produit ne doit être confondu avec un autre et chaque implémentation doit rester cohérente avec les règles transverses de sécurité, audit, configuration et multi-pays.

## Produits obligatoires

1. **Application mobile Client** — portefeuille, cartes, virements, QR/NFC, Mobile Money, budgets, coffres, abonnements, fidélité, reçus, support et assistant Jini.
2. **Application mobile Commerçant** — encaissement, catalogue, mini-site, factures, fidélité, promotions, employés, rapports, rapprochement, litiges et support.
3. **Application TPE Android** — paiement carte/QR/NFC, mode dégradé contrôlé, partage d’addition, remboursement, reçus, clôture, synchronisation et gestion distante.
4. **Application Admin Lite mobile** — supervision et actions urgentes soumises à permissions, journalisation et double validation selon criticité.
5. **Application mobile Annuaire/Hub** — recherche locale, géolocalisation, catégories, profils professionnels, mini-sites, promotions, favoris, avis, itinéraires, réservation ou prise de contact, paiement Mansa et monétisation.
6. **Site web public officiel Mansa** — présentation institutionnelle, fonctionnalités, chiffres clés, sécurité, tarifs, partenaires, actualités, carrières, aide, téléchargements, pages légales, SEO, analytics et CMS.
7. **Site web professionnel/commerçants** — offres TPE, solutions métiers, onboarding, tarification, démonstrations, documentation commerciale, espace partenaire, formulaires, leads, mini-sites et fidélité.
8. **Portail Admin Web** — contrôle complet de la plateforme, des contenus, des risques, des partenaires, des environnements et des configurations.
9. **Backend/API Gateway** — authentification, orchestration, règles métier, webhooks, intégrations et séparation Démo/Recette/Production.
10. **Services IA** — Jini, aide contextuelle, détection de fraude, support et recommandations, avec supervision humaine et traçabilité.
11. **Packages partagés** — design system, contrats, schémas, observabilité, sécurité, configuration et clients API.
12. **Infrastructure et CI/CD** — validation, déploiement, secrets externes, observabilité, sauvegardes, reprise, haute disponibilité et conformité.

## Interfaces web distinctes

Les trois interfaces suivantes sont séparées :

- `web-public` : communication au grand public et aux utilisateurs.
- `web-business` : acquisition, vente et accompagnement des commerçants et partenaires.
- `admin-web` : administration interne, opérations, conformité, risque et pilotage.

Elles peuvent partager le design system et le moteur d’animations, mais leurs parcours, droits, données et objectifs restent indépendants.

## Règles transverses obligatoires

- Tout paramètre métier important doit être administrable avec historique des changements.
- Toute action sensible doit produire un audit log non ambigu : acteur, rôle, cible, avant/après, motif, date, environnement et corrélation technique.
- Les opérations critiques exigent une double validation configurable.
- Les intégrations externes passent par des adaptateurs isolés.
- Aucun secret réel n’est stocké dans Git.
- Les fonctions doivent supporter plusieurs pays, devises, langues et partenaires.
- Les environnements Démo, Recette et Production sont strictement séparés.
- Les fonctions indisponibles ou incomplètes doivent être masquées derrière des feature flags.
- Les décisions IA à impact financier ou réglementaire doivent être explicables et révisables.

## Éléments historiques à préserver

- Cartes physiques, virtuelles, temporaires et jetables.
- Détection et gestion des abonnements.
- Coffres et objectifs d’épargne.
- QR, NFC, Wallets mobiles et Mobile Money.
- Paiement partagé ou séparé au restaurant et dans tout commerce adapté.
- TPE de démonstration et mode Démo/Recette/Production.
- Module État : amendes, taxes, scolarité, bourses, cartes étudiantes, identification agent et lutte anticorruption.
- Annuaire avec abonnements sectoriels, géolocalisation, profils mis en avant et mini-sites.
- Administration entièrement configurable avec blocage réel des fonctions désactivées.

## Données manquantes

Les formulations exactes qui n’existent que dans un export DeepSeek non présent dans le dépôt doivent être ajoutées sous la mention :

> À compléter depuis l’export DeepSeek

Elles ne doivent jamais être inventées ni présentées comme déjà validées.