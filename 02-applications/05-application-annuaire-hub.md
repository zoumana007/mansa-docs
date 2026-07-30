# Application mobile Annuaire / Hub

## 1. Positionnement

L’Annuaire / Hub est une application mobile autonome. Elle n’est ni un onglet secondaire de l’application Client, ni une copie de l’application Commerçant. Elle permet de découvrir des professionnels, services, commerces, institutions et offres locales, puis d’entrer en relation, réserver, demander un devis ou payer avec Mansa selon le secteur.

## 2. Utilisateurs et rôles

- Visiteur non connecté.
- Utilisateur particulier.
- Professionnel propriétaire d’une fiche.
- Employé d’un professionnel.
- Modérateur.
- Administrateur annuaire.
- Super administrateur.

## 3. Écrans principaux

Accueil personnalisé, carte, recherche, catégories, résultats, filtres, fiche professionnelle, galerie, offres, promotions, avis, favoris, itinéraire, contact, demande de devis, réservation, paiement, historique, notifications, profil utilisateur, espace professionnel, statistiques et paramètres.

## 4. Recherche et découverte

Recherche textuelle tolérante, catégories, sous-catégories, distance, commune, horaires, disponibilité, note, prix indicatif, promotion, établissement vérifié, paiement Mansa accepté et accessibilité. La géolocalisation est facultative ; l’utilisateur peut choisir manuellement une zone.

## 5. Fiche professionnelle et mini-site

Chaque professionnel dispose d’une fiche pouvant générer un mini-site public administrable : identité, description, adresse, coordonnées, horaires, galerie, catalogue ou services, tarifs indicatifs, équipe, moyens de paiement, liens, certifications, offres, avis et boutons d’action. Les champs visibles dépendent du secteur.

## 6. Secteurs et comportements configurables

L’administration définit les secteurs, leurs champs, filtres et actions : commerce, restaurant, artisan, bâtiment, santé, éducation, transport, événementiel, tourisme, services administratifs et autres. Une fiche peut proposer contact simple, réservation, créneau, commande, demande de devis ou paiement, sans forcer le même parcours à tous les métiers.

## 7. Abonnements et monétisation

Plans gratuits et payants configurables : visibilité renforcée, galerie étendue, statistiques, offres, domaine ou mini-site, badge, emplacement à la une et outils de conversion. Les classements sponsorisés sont clairement identifiés. Les commissions, abonnements, périodes d’essai et taxes sont administrables par pays et secteur.

## 8. Avis et modération

Avis liés à un compte ; possibilité d’exiger une interaction ou transaction vérifiée selon politique. Signalement, réponse professionnelle, masquage temporaire, décision motivée, appel et journal d’audit. Détection anti-spam et limites de fréquence.

## 9. Réservation, devis et contact

Créneaux, capacité, ressources, confirmation, annulation, rappel, acompte et remboursement configurables. Pour les secteurs non réservables, demande de devis structurée avec pièces jointes contrôlées. Les coordonnées privées sont protégées et le consentement est enregistré.

## 10. Paiement Mansa

Paiement complet, acompte, lien, QR ou paiement sur place selon configuration. Chaque opération utilise idempotence, reçu, statut fiable, remboursement autorisé et rapprochement avec le compte professionnel. L’annuaire ne manipule jamais directement les soldes : il appelle les services financiers de la plateforme.

## 11. Espace professionnel

Gestion de fiche, établissements multiples, employés et permissions, contenus, horaires, catalogue, offres, leads, devis, réservations, paiements, avis, statistiques, abonnement, factures et support. Les changements sensibles peuvent nécessiter validation ou vérification KYB.

## 12. Administration

Catégories, champs dynamiques, zones, règles de classement, contenus à la une, abonnements, commissions, modération, signalements, avis, badges, fiches, professionnels, campagnes, statistiques, fraude, traductions et feature flags. Toute intervention est tracée.

## 13. API et données

Entités minimales : DirectoryProfile, Establishment, Category, CategoryField, Location, OpeningHours, Media, ServiceItem, Offer, Promotion, Favorite, Review, ReviewReport, Booking, BookingSlot, QuoteRequest, DirectorySubscription, FeaturedPlacement, DirectoryLead et DirectoryAnalyticsEvent.

Familles d’API : recherche, géospatial, profils, médias, catalogue, offres, favoris, avis, signalements, réservation, devis, paiement, abonnement, analytics, modération et administration. Les endpoints d’écriture exigent authentification, autorisation et validation stricte.

## 14. Sécurité et confidentialité

Permission explicite pour localisation, précision réduite lorsque suffisante, masquage des coordonnées sensibles, contrôle des fichiers, anti-abus, limitation de débit, détection d’automatisation, chiffrement, audit et rétention configurable. Un professionnel ne voit que les données nécessaires au traitement d’une demande.

## 15. Tests et critères d’acceptation

- L’application est déployable indépendamment.
- Recherche et filtres fonctionnent sans localisation obligatoire.
- Les fiches s’adaptent au secteur configuré.
- Réservation et devis ne sont activés que lorsque compatibles.
- Les paiements produisent un reçu et un statut vérifiable.
- Les placements sponsorisés sont identifiables.
- Les permissions professionnelles sont cloisonnées.
- Les avis et signalements possèdent un workflow complet.
- Les écrans critiques ont tests unitaires, intégration et e2e.
- Les performances restent acceptables sur appareils et réseaux modestes.