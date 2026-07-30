# Application mobile Annuaire / Hub

## Positionnement

L’Annuaire est une application autonome, distincte des applications Client et Commerçant. Elle met en relation utilisateurs, professionnels, commerces et services locaux tout en utilisant l’identité et les paiements Mansa lorsque cela apporte une valeur claire.

## Utilisateurs et rôles

- Visiteur : recherche et consultation publiques limitées.
- Utilisateur connecté : favoris, avis, contacts, réservations et paiements.
- Professionnel : revendication et gestion de profil, offres, statistiques et abonnement.
- Modérateur : contrôle des contenus, avis, signalements et catégories.
- Administrateur Annuaire : configuration, monétisation, visibilité et supervision.

## Fonctionnalités utilisateur

Recherche plein texte, catégories, sous-catégories, filtres, carte, proximité, ville/quartier, horaires, accessibilité, prix, note et disponibilité. Les résultats distinguent clairement le classement naturel, le contenu sponsorisé et les établissements à la une.

Chaque profil professionnel comprend identité, vérification, description, médias, horaires, adresses, contacts, services, catalogue, promotions, moyens de paiement, itinéraire, liens, avis, FAQ et actions adaptées au secteur.

L’utilisateur peut enregistrer des favoris, partager, signaler, appeler, écrire, demander un devis, prendre rendez-vous ou réserver lorsque le secteur le permet. Le produit ne force pas un modèle de réservation unique pour tous les métiers.

## Mini-sites automatiques

Un professionnel validé peut publier un mini-site à partir de son profil. Le mini-site possède URL stable, thème, contenus, galerie, catalogue, offres, formulaire, analytics, SEO local et bouton de paiement Mansa. Les composants sont administrables mais restent dans un cadre sécurisé.

## Abonnements sectoriels et monétisation

Plans configurables par pays et secteur : gratuit, vérifié, professionnel, premium ou campagne. Les avantages peuvent inclure statistiques avancées, plus de médias, offres, mini-site, priorité contrôlée, leads et outils de conversion. Les règles de classement sponsorisé sont transparentes et auditables.

## Avis et modération

Un avis nécessite un compte et peut être renforcé par une preuve d’interaction ou de paiement. Détection de spam, signalement, réponse du professionnel, droit de recours, historique des décisions et modération humaine. Aucun avis ne doit disparaître uniquement parce qu’il est négatif.

## Paiement Mansa

Paiement d’acompte, commande, réservation ou facture selon le secteur. L’Annuaire ne modifie jamais directement un solde : il appelle les API paiement, reçoit un statut et affiche le résultat. Idempotence, reçu, remboursement et litige suivent les règles générales de Mansa.

## Géolocalisation et confidentialité

La position précise est optionnelle, demandée au moment utile et jamais nécessaire pour une recherche par ville. Stockage minimal, précision réduite lorsque possible, suppression des historiques et explication claire de l’usage.

## Écrans minimum

Onboarding, accueil, catégories, recherche, carte, résultats, profil professionnel, offres, mini-site, favoris, avis, réservation/contact, paiement, notifications, compte, espace professionnel, statistiques, abonnement, modération et paramètres.

## API et données

Entités principales : DirectoryCategory, DirectoryListing, ListingLocation, ListingSchedule, ListingMedia, ListingService, ListingOffer, ListingSubscription, DirectoryReview, ReviewEvidence, Favorite, Lead, Booking, DirectoryPaymentReference, ModerationCase, SponsoredPlacement et DirectoryMetric.

API versionnée pour recherche, géocodage abstrait, profils, médias, offres, avis, favoris, leads, réservations, abonnements, paiements, statistiques et modération. Pagination curseur pour grands ensembles et cache invalidé par événements.

## Administration

Gestion des catégories, secteurs, attributs dynamiques, vérification, plans, prix, règles de visibilité, campagnes, contenus à la une, litiges, avis, signalements, exports et métriques. Toute modification de classement ou de monétisation est tracée.

## Sécurité

Contrôle d’accès par rôle et propriété, validation stricte des médias, antivirus, quotas, rate limiting, masquage de données, prévention du scraping abusif, audit et protections contre faux avis, prise de contrôle de profil et redirections malveillantes.

## Critères d’acceptation

1. L’application fonctionne indépendamment de l’application Client.
2. Une recherche sans géolocalisation reste possible.
3. Le contenu sponsorisé est toujours identifiable.
4. Un professionnel ne peut modifier qu’un profil qu’il contrôle.
5. Un paiement en échec ne crée ni réservation confirmée ni débit fantôme.
6. Les avis suivent une procédure de modération et de recours traçable.
7. Les mini-sites sont responsive, accessibles et isolés des scripts arbitraires.
8. Les statistiques ne révèlent aucune donnée personnelle d’un utilisateur.
9. Les règles de visibilité sont configurables depuis l’Admin Web.
10. Les secteurs sans réservation utilisent une prise de contact adaptée plutôt qu’un parcours artificiel.