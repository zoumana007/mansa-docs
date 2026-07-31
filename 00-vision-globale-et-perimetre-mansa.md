# Vision globale et périmètre officiel de Mansa

## 1. Rôle du présent document

Ce document définit le périmètre officiel de l’écosystème **Mansa**.

Il sert de référence principale pour :

- la conception fonctionnelle ;
- la conception technique ;
- le développement par Codex ou une autre IA ;
- le travail des futures équipes ;
- la création des applications, sites web, API et services ;
- la vérification qu’aucune partie du projet n’a été oubliée.

En cas de contradiction entre un ancien prompt, une ancienne réponse d’IA et la documentation actuelle, la version validée dans `mansa-docs` devient la référence.

Le dépôt `mansa-docs` contient les spécifications.

Le dépôt `mansa-platform` contient le code source réel.

## 2. Vision de Mansa

Mansa est un écosystème fintech modulaire conçu initialement pour le Mali, puis pour d’autres pays africains.

La plateforme doit réunir dans un même environnement :

- paiements ;
- comptes et wallets ;
- cartes ;
- Mobile Money ;
- services commerçants ;
- TPE ;
- gestion d’entreprise ;
- annuaire professionnel ;
- services publics ;
- taxes et amendes ;
- cartes étudiantes ;
- bourses ;
- investissements ;
- intelligence artificielle ;
- fidélité ;
- facturation ;
- reçus numériques ;
- remboursements ;
- services partenaires.

L’objectif n’est pas de créer une simple application de paiement.

Mansa doit devenir un **écosystème financier, commercial, institutionnel et technologique complet**, composé de plusieurs applications indépendantes mais interconnectées.

Chaque produit devra viser une expérience simple, moderne, fiable, fortement intégrée et réellement différenciante. Les futurs chapitres fonctionnels devront détailler les parcours, règles, écrans, automatisations, personnalisation, sécurité, administration, cas limites et critères d’acceptation avec un niveau d’ambition élevé, afin que Mansa ne soit pas une copie minimale d’une fintech existante mais un produit innovant adapté aux réalités africaines.

## 3. Principes fondamentaux

Tous les produits Mansa doivent respecter les principes suivants.

### 3.1 Modularité

Chaque application et chaque module doit pouvoir évoluer séparément.

Une fonctionnalité doit pouvoir être :

- activée ;
- désactivée ;
- limitée ;
- déplacée vers un autre abonnement ;
- réservée à certains rôles ;
- réservée à certains pays ;
- réservée à certains partenaires ;

sans devoir modifier toute la plateforme.

### 3.2 Administration sans modification du code

L’administration centrale doit permettre, dans la mesure du possible, de modifier sans toucher au code :

- le nom des fonctionnalités ;
- leur visibilité ;
- leur ordre d’affichage ;
- leur disponibilité ;
- les abonnements ;
- les limites ;
- les commissions ;
- les frais ;
- les pays ;
- les devises ;
- les langues ;
- les rôles ;
- les permissions ;
- les partenaires ;
- les modèles de reçus ;
- les contenus des sites web ;
- les promotions ;
- les catégories de l’annuaire ;
- les paramètres des services publics.

Certaines modifications techniques profondes nécessiteront néanmoins une nouvelle version du code.

### 3.3 Traçabilité

Toute action sensible doit être enregistrée :

- auteur ;
- date ;
- heure ;
- appareil ;
- adresse IP lorsque disponible ;
- ancienne valeur ;
- nouvelle valeur ;
- motif ;
- approbateur éventuel.

Les journaux critiques doivent être protégés contre la modification non autorisée.

### 3.4 Sécurité par défaut

La plateforme doit appliquer notamment :

- authentification forte ;
- double authentification ;
- biométrie lorsque disponible ;
- RBAC ;
- permissions fines ;
- chiffrement ;
- gestion sécurisée des sessions ;
- journalisation ;
- détection des comportements suspects ;
- limitations de débit ;
- protections contre les attaques courantes ;
- séparation Démo, Recette et Production ;
- aucune clé secrète dans GitHub.

### 3.5 Multi-pays

Mansa doit être conçu pour gérer :

- plusieurs pays ;
- plusieurs devises ;
- plusieurs langues ;
- plusieurs formats de téléphone ;
- plusieurs règles fiscales ;
- plusieurs partenaires bancaires ;
- plusieurs opérateurs Mobile Money ;
- plusieurs réglementations.

### 3.6 Partenaires externes

Certaines fonctionnalités ne peuvent pas être exploitées légalement ou techniquement sans partenaire.

Cela concerne notamment :

- émission de cartes ;
- acquisition bancaire ;
- Visa ;
- Mastercard ;
- Mobile Money ;
- Tap to Phone ;
- investissements ;
- levées de fonds ;
- services publics ;
- taxes ;
- amendes ;
- cartes étudiantes ;
- bourses ;
- Apple Wallet ;
- Google Wallet.

Mansa doit prévoir l’architecture et les interfaces, mais l’activation réelle dépendra des contrats, licences et autorisations nécessaires.

## 4. Produits officiels de l’écosystème

### 4.1 Application mobile Mansa Client

Application destinée aux particuliers.

Fonctions principales prévues :

- inscription ;
- authentification ;
- KYC ;
- wallet ;
- comptes ;
- solde ;
- historique ;
- transferts ;
- paiements ;
- QR Code ;
- NFC ;
- cartes physiques ;
- cartes virtuelles ;
- carte jetable ;
- blocage et déblocage ;
- budgets ;
- coffres d’épargne ;
- partage de dépenses ;
- centre familial ;
- notifications ;
- sécurité ;
- appareils connectés ;
- reçus numériques ;
- garanties ;
- remboursements ;
- fidélité ;
- services publics ;
- investissements lorsque autorisés ;
- assistant Jini.

L’application Client ne doit pas contenir toute la gestion professionnelle des commerçants.

Cette liste est uniquement le périmètre général. Le futur dossier consacré à l’application Client devra détailler intégralement les fonctionnalités, écrans, parcours, règles métier, automatisations, niveaux de compte, sécurité, personnalisation, expériences familiales et jeunes, budgets, coffres, cartes, paiements, reçus, garanties, remboursements, investissements, services publics, IA, accessibilité, notifications et cas limites. L’objectif est de concevoir une application client ambitieuse, innovante et cohérente, et non une simple interface de wallet.

### 4.2 Application mobile Mansa Commerce

Application destinée aux commerçants, entreprises et professionnels.

Fonctions principales prévues :

- gestion du commerce ;
- employés ;
- rôles ;
- permissions ;
- ventes ;
- produits ;
- services ;
- catalogue ;
- stocks ;
- inventaire ;
- codes-barres ;
- QR produits ;
- génération d’étiquettes ;
- images produits ;
- fournisseurs ;
- clients ;
- reçus ;
- factures ;
- taxes ;
- remboursements ;
- retours ;
- échanges ;
- promotions ;
- campagnes ;
- fidélité ;
- cadeaux ;
- rendez-vous selon le secteur ;
- mini-site professionnel ;
- statistiques ;
- rapports ;
- impression thermique ;
- configuration des abonnements ;
- connexion avec le TPE ;
- connexion avec l’Annuaire/Hub.

L’application Commerce doit fonctionner pour plusieurs secteurs, pas seulement les restaurants.

### 4.3 Application Mansa TPE Android

Application de paiement destinée aux terminaux Android professionnels et, lorsque les autorisations le permettent, aux téléphones compatibles.

Fonctions principales prévues :

- saisie du montant ;
- catalogue ;
- scan produit ;
- suppression d’un article déjà scanné ;
- modification de quantité ;
- remise ;
- taxes ;
- paiement par carte ;
- NFC ;
- QR ;
- Mobile Money ;
- Tap to Phone ;
- paiement fractionné ;
- partage d’addition ;
- remboursement ;
- annulation ;
- reçu numérique ;
- reçu imprimé ;
- imprimantes Bluetooth, Wi-Fi, USB ou intégrées ;
- rapports de caisse ;
- mode hors ligne limité ;
- synchronisation ;
- gestion des employés ;
- sécurité du terminal ;
- mode Démo, Recette et Production.

L’application TPE est distincte de l’application Commerce, même si elles partagent certaines données.

### 4.4 Application mobile Mansa Admin Lite

Application destinée aux administrateurs autorisés qui doivent effectuer certaines tâches depuis un téléphone.

Fonctions possibles :

- alertes ;
- validation limitée ;
- incidents ;
- support ;
- consultation de tableaux de bord ;
- blocage d’urgence ;
- surveillance ;
- approbation selon permissions ;
- notifications critiques.

Admin Lite ne doit pas remplacer le portail Admin Web complet.

### 4.5 Application mobile Mansa Annuaire / Hub

Application distincte, destinée au public et aux professionnels.

Fonctions principales prévues :

- recherche de commerces ;
- recherche de services ;
- catégories ;
- géolocalisation ;
- carte ;
- itinéraires ;
- appel ;
- e-mail ;
- WhatsApp ;
- profils professionnels ;
- mini-sites ;
- horaires ;
- photos ;
- produits ;
- services ;
- promotions ;
- favoris ;
- avis ;
- modération ;
- prise de rendez-vous ;
- réservation selon le secteur ;
- disponibilité en temps réel ;
- établissements à la une ;
- abonnements sectoriels ;
- paiements Mansa ;
- statistiques professionnelles.

Un utilisateur peut consulter l’annuaire sans abonnement professionnel.

Un professionnel doit disposer d’un abonnement compatible dans Mansa Commerce ou souscrire séparément pour publier et gérer son profil.

## 5. Interfaces web officielles

### 5.1 Site web public officiel Mansa

Site institutionnel et commercial destiné au grand public.

Il doit présenter :

- la vision ;
- les produits ;
- les applications ;
- les cartes ;
- les paiements ;
- la sécurité ;
- les partenaires ;
- les chiffres clés ;
- le nombre d’utilisateurs ;
- l’impact ;
- les tarifs ;
- les actualités ;
- les carrières ;
- l’aide ;
- les téléchargements ;
- les pages légales ;
- les contacts ;
- les fonctionnalités futures.

Le contenu doit être administrable depuis le CMS.

Le site doit avoir un rendu premium, moderne, fluide et performant.

### 5.2 Site web Mansa Professionnels

Site séparé destiné aux :

- commerçants ;
- entreprises ;
- partenaires ;
- banques ;
- institutions ;
- développeurs professionnels.

Il doit présenter :

- offres commerçants ;
- offres entreprises ;
- TPE ;
- Tap to Phone ;
- paiements ;
- abonnements ;
- fidélité ;
- promotions ;
- catalogue ;
- stocks ;
- mini-sites ;
- annuaire ;
- démonstrations ;
- demande de devis ;
- inscription ;
- accompagnement ;
- documentation commerciale ;
- partenariats ;
- formulaires ;
- espace professionnel.

Son contenu doit également être administrable.

### 5.3 Portail Mansa Admin Web

Interface centrale de gestion de tout l’écosystème.

Elle doit permettre de gérer :

- utilisateurs ;
- commerçants ;
- entreprises ;
- employés ;
- rôles ;
- permissions ;
- KYC ;
- KYB ;
- wallets ;
- comptes ;
- cartes ;
- transactions ;
- paiements ;
- transferts ;
- TPE ;
- imprimantes ;
- remboursements ;
- litiges ;
- fraude ;
- support ;
- notifications ;
- abonnements ;
- commissions ;
- promotions ;
- catalogue ;
- annuaire ;
- contenus des deux sites ;
- partenaires ;
- banques ;
- Mobile Money ;
- pays ;
- devises ;
- langues ;
- module État ;
- taxes ;
- amendes ;
- bourses ;
- cartes étudiantes ;
- investissements ;
- Jini ;
- statistiques ;
- rapports ;
- audit ;
- incidents ;
- feature flags ;
- environnements ;
- configuration générale.

Les actions critiques doivent pouvoir exiger une double validation.

Le portail Admin doit privilégier l’efficacité, la lisibilité et la sécurité. Les animations premium ne doivent jamais gêner les opérations.

## 6. Socle technique officiel

### 6.1 Backend et API Gateway

Le backend central doit :

- exposer les API ;
- gérer l’authentification ;
- appliquer les permissions ;
- orchestrer les modules ;
- connecter les partenaires ;
- gérer les erreurs ;
- garantir l’idempotence ;
- journaliser les actions ;
- publier les événements ;
- gérer les tâches asynchrones ;
- supporter plusieurs applications.

Stack prévue :

- NestJS ;
- TypeScript ;
- PostgreSQL ;
- Prisma ;
- Redis lorsque nécessaire ;
- files de messages lorsque nécessaire.

### 6.2 Services IA Jini

Jini est l’assistant IA de Mansa.

Il pourra intervenir dans :

- assistance utilisateur ;
- support ;
- aide financière ;
- explication des dépenses ;
- recommandations ;
- recherche ;
- aide commerçant ;
- détection de fraude ;
- analyse ;
- aide administrative.

Il ne doit jamais prendre seul une décision financière ou administrative critique sans règles, permissions et supervision adaptées.

### 6.3 Packages partagés

Le monorepo doit prévoir des packages partagés pour :

- types ;
- contrats ;
- SDK ;
- utilitaires ;
- design system ;
- composants UI ;
- authentification ;
- permissions ;
- paiements ;
- notifications ;
- configuration ;
- thèmes ;
- internationalisation ;
- analytics.

### 6.4 Infrastructure

Le projet doit prévoir :

- CI ;
- tests ;
- formatage ;
- lint ;
- build ;
- déploiement ;
- monitoring ;
- alertes ;
- sauvegardes ;
- journaux ;
- gestion des environnements ;
- gestion externe des secrets ;
- documentation d’exploitation.

## 7. Design et expérience premium

Les trois interfaces web doivent viser un niveau de finition comparable aux meilleurs produits numériques modernes, sans copier directement une marque existante.

Technologies envisagées :

- Next.js App Router ;
- React ;
- TypeScript ;
- Tailwind CSS ;
- Framer Motion ;
- GSAP ;
- ScrollTrigger ;
- Lenis ;
- Three.js ;
- React Three Fiber ;
- Drei ;
- Blender ;
- Spline uniquement si utile.

Exigences :

- animations fluides ;
- storytelling au scroll ;
- parallax ;
- micro-interactions ;
- transitions de pages ;
- smartphones 3D ;
- TPE 3D ;
- éclairage ;
- reflets ;
- loaders ;
- responsive ;
- accessibilité ;
- `prefers-reduced-motion` ;
- fallbacks 2D ;
- optimisation CPU et GPU ;
- chargement différé ;
- limitation des re-rendus.

Les performances et l’accessibilité restent prioritaires sur les effets visuels.

## 8. Répartition entre les dépôts

### `mansa-docs`

Contient :

- vision ;
- spécifications ;
- prompts ;
- règles métier ;
- contrats API ;
- schémas ;
- critères d’acceptation ;
- parcours ;
- décisions d’architecture ;
- exigences de tests ;
- documentation de déploiement.

### `mansa-platform`

Contient principalement :

- code source ;
- tests ;
- configurations ;
- migrations ;
- scripts ;
- CI ;
- exemples d’environnement ;
- documentation technique minimale nécessaire à l’exécution.

Le dépôt plateforme ne doit pas être rempli de longs textes fonctionnels déjà présents dans `mansa-docs`.

## 9. Statuts des fonctionnalités

Chaque fonctionnalité documentée doit recevoir un statut :

- `À confirmer`
- `Validée`
- `Prévue`
- `En développement`
- `Disponible en démonstration`
- `Dépend d’un partenaire`
- `Dépend d’une autorisation`
- `Disponible en production`
- `Suspendue`
- `Abandonnée`

Une fonctionnalité ne doit jamais être présentée comme opérationnelle simplement parce qu’une interface ou un exemple de code existe.

## 10. Règle de validation

Aucun fichier n’est considéré comme définitif tant qu’il n’a pas été validé par le propriétaire du projet.

Avant chaque ajout important dans GitHub :

1. le contenu est présenté ;
2. les corrections sont demandées ;
3. le propriétaire valide ;
4. le fichier est poussé ;
5. le commit est communiqué.
