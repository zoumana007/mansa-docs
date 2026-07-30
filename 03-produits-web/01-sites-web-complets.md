# Les deux sites web Mansa — spécification consolidée

## A. Site public officiel Mansa

### Objectif
Présenter Mansa au grand public, rassurer, convertir vers les applications et publier des informations administrables sans déploiement technique.

### Pages minimales
Accueil, vision, particuliers, cartes, paiements, coffres et budgets, sécurité, tarifs, partenaires, État et impact, chiffres clés, actualités, presse, carrières, centre d’aide, contact, téléchargement, statut des services, mentions légales, confidentialité, cookies et conditions.

### Blocs de l’accueil
Hero animé, proposition de valeur, démonstration produit, chiffres configurables, applications disponibles, sécurité, cas d’usage, partenaires, témoignages modérés, couverture géographique, appel au téléchargement et pied de page complet.

### CMS et administration
Tous les textes, visuels, vidéos, statistiques, liens, tarifs, partenaires, pays, langues, métadonnées SEO et blocs de page sont versionnés et modifiables depuis Admin Web. Prévisualisation, brouillon, planification, publication, retour arrière et journal d’audit sont obligatoires.

### Design et animations
Next.js, rendu rapide, responsive, accessibilité WCAG, animations Framer Motion ou GSAP selon besoin, parallaxe légère, scènes 3D uniquement si elles restent performantes, mode réduction des mouvements, chargement progressif et aucune animation bloquante.

### Données et API
Endpoints publics en lecture pour contenu publié, tarifs, disponibilité pays, liens d’applications, FAQ, actualités et statut. Formulaires protégés contre spam avec consentement, quotas et stockage minimal.

### Analytics
Événements de campagne, téléchargements, clics partenaires, formulaires, parcours et conversions. Consentement préalable selon juridiction. Aucun secret financier ni donnée utilisateur privée ne doit être rendu public.

### Critères d’acceptation
- Contenu modifiable sans changement de code.
- Page principale utilisable sur réseau lent.
- SEO, sitemap, robots, Open Graph et données structurées.
- Accessibilité clavier et lecteur d’écran.
- Chiffres clés datés, sourcés et désactivables.
- Tests unitaires, composants, e2e et performance.

## B. Site Commerçants & Partenaires

### Objectif
Acquérir, informer et convertir commerçants, entreprises, intégrateurs, banques et partenaires institutionnels.

### Pages minimales
Accueil professionnel, encaissement, TPE, QR et liens, application Commerçant, mini-sites, catalogue, fidélité, promotions, partage d’addition, facturation, tarifs, matériel, intégrations, partenaires, cas clients, ressources, FAQ, démonstration, demande de devis, inscription et espace partenaire.

### Parcours principaux
1. Prospect découvre une offre, simule ses besoins, demande une démonstration et devient lead.
2. Commerçant démarre l’onboarding, crée l’entreprise, fournit le KYB, choisit une offre et suit l’état du dossier.
3. Partenaire technique consulte la documentation autorisée et demande des identifiants sandbox.
4. Partenaire institutionnel soumet une demande structurée et suit les échanges dans son espace.

### TPE et matériel
Catalogue administrable, caractéristiques, compatibilités, disponibilité par pays, location ou achat selon configuration, demande de matériel, état de commande et association ultérieure à un compte commerçant.

### Espace partenaire
Authentification forte, demandes, documents, contrats, tickets, sandbox, webhooks, rapports autorisés et gestion des membres. Les permissions sont séparées par organisation.

### Leads et CRM
Formulaires configurables, source de campagne, attribution, consentement, statut, propriétaire, notes, tâches, export contrôlé et intégration via événement/API avec un CRM futur.

### CMS, analytics et sécurité
Même socle de publication que le site public, avec contenus professionnels séparés. Protection anti-abus, validation des fichiers, scan antivirus, limitation de débit, journaux d’audit et politiques de rétention.

### Critères d’acceptation
- Site distinct du site public mais design system partagé.
- Onboarding reprenable sans perte.
- Tarifs et offres configurables par pays et segment.
- Leads visibles dans Admin Web avec source et consentement.
- Espace partenaire cloisonné par organisation.
- Tests e2e des formulaires, onboarding et permissions.

## Administration commune
Admin Web gère domaines, navigation, redirections, traductions, composants, médias, formulaires, consentements, campagnes, publications, SEO, analytics et incidents. Toute modification sensible précise auteur, date, ancienne valeur et nouvelle valeur.