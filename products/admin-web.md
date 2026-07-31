# Portail Admin Web Mansa

## Mission

Le portail Admin Web est le centre de contrôle de la plateforme. Il ne doit pas être une simple collection de tableaux : il doit fournir une gouvernance complète, des permissions fines, une traçabilité exploitable et des workflows de validation adaptés à la criticité.

## Rôles principaux

- Super administrateur de plateforme.
- Administrateur pays.
- Opérations paiements.
- Conformité KYC/KYB.
- Risque et fraude.
- Support client.
- Support commerçant.
- Finance et rapprochement.
- Gestionnaire TPE.
- Gestionnaire CMS.
- Gestionnaire Annuaire/Hub.
- Administrateur services publics.
- Auditeur en lecture seule.
- Responsable sécurité.

Les rôles combinent RBAC et attributs ABAC : pays, entité, partenaire, environnement, montant, type de ressource, niveau de sensibilité et horaire autorisé.

## Navigation fonctionnelle

1. Dashboard global.
2. Utilisateurs et identité.
3. Commerçants et KYB.
4. Wallets et comptes.
5. Transactions et paiements.
6. Cartes.
7. TPE et parc matériel.
8. Mobile Money et banques.
9. Fraude, risque et limites.
10. Litiges, remboursements et chargebacks.
11. Support et communication.
12. CMS site public.
13. CMS site professionnel.
14. Annuaire/Hub.
15. Module État.
16. Bourses et cartes étudiantes.
17. Taxes et amendes.
18. IA/Jini.
19. Tarification et commissions.
20. Pays, devises et langues.
21. Partenaires et intégrations.
22. Feature flags et configuration distante.
23. Analytics et rapports.
24. Monitoring et incidents.
25. Audit et conformité.
26. Paramètres de sécurité.

## Dashboard

Le dashboard présente des indicateurs temps réel ou quasi temps réel selon leur source : volumes, valeurs, taux de succès, incidents, fraude, KYC en attente, disponibilité partenaires, parc TPE, support, liquidité et rapprochements. Chaque chiffre affiche sa source, sa période, son fuseau horaire et son niveau de fraîcheur.

## Utilisateurs, KYC et appareils

Fonctions : recherche sécurisée, profil, statut, documents masqués par défaut, consentements, appareils, sessions, limites, bénéficiaires, historique, alertes et actions contrôlées. Toute consultation de donnée sensible peut être auditée. Les documents KYC ne sont jamais téléchargeables sans permission dédiée.

## Commerçants et KYB

Gestion des entreprises, établissements, bénéficiaires effectifs, employés, rôles, caisses, catalogues, mini-sites, commissions, TPE, règlements, réserves, litiges et indicateurs. Les changements de compte bancaire ou bénéficiaire nécessitent une vérification renforcée.

## Transactions

Recherche par identifiant, référence partenaire, utilisateur, commerçant, terminal, période, statut, canal et montant. La fiche transaction expose sa chronologie, ses écritures de ledger, frais, événements, webhooks, rapprochement, risques et actions autorisées. Aucune modification directe d’une transaction finalisée n’est permise.

## Cartes et wallets

Gestion du cycle de vie des cartes physiques, virtuelles et temporaires : demande, production, activation, gel, remplacement, limites et fermeture. Les données PCI sensibles ne sont pas stockées ni affichées hors des composants autorisés du partenaire. Les wallets exposent leurs soldes disponibles, comptables, réservés et leurs écritures.

## TPE

Inventaire, association, certificats, version logicielle, statut, diagnostics, dernière synchronisation, commerçant, emplacement, configuration, limites, mises à jour et révocation. Toute commande distante est signée, auditée et soumise à permission.

## Fraude et risque

Règles, alertes, dossiers, signaux, score, historique, décisions, listes, limites et revue manuelle. Une décision automatique importante doit être explicable. Les règles disposent de versions, tests, approbations et rollback.

## Support et litiges

Tickets, conversations, SLA, pièces jointes contrôlées, catégories, escalades, remboursement, contestation et résolution. Les agents ne peuvent pas initier une opération financière au-delà de leurs limites et sans workflow adapté.

## CMS des deux sites

Gestion séparée du site public et du site professionnel : pages, composants, actualités, FAQ, tarifs, chiffres clés, partenaires, carrières, formulaires, SEO, traductions, médias, brouillons, prévisualisation, planification, approbation et publication. Les chiffres financiers ou d’impact nécessitent une validation.

## Annuaire/Hub

Catégories, secteurs, profils professionnels, mini-sites, offres, promotions, abonnements, mise en avant, avis, modération, signalements, réservations, règles de visibilité, tarification et statistiques.

## Module État

Gestion des organismes, agents, terminaux, référentiels, infractions, taxes, services, établissements, étudiants, cartes, bourses, règlements, rapprochements et audits. Les suppressions destructives sont interdites pour les données réglementaires ; utiliser statuts, corrections et écritures compensatoires.

## IA/Jini

Gestion des fonctionnalités autorisées, modèles, versions de prompts système, bases documentaires, seuils, évaluations, journaux, coûts et incidents. Les données personnelles sont minimisées et les réponses sensibles peuvent nécessiter validation humaine.

## Configuration et environnements

Démo, Recette et Production sont visuellement distincts et techniquement séparés. Les changements de production utilisent un workflow de demande, revue, approbation et application. Les secrets sont référencés depuis un gestionnaire externe et jamais révélés dans l’interface.

## Audit et double validation

Chaque action sensible enregistre acteur, rôle, ressource, ancienne valeur, nouvelle valeur, justification, adresse ou contexte technique, horodatage, corrélation et résultat. Les opérations critiques créent une demande distincte qu’un second acteur autorisé doit approuver.

Exemples : changement de commission globale, modification de limites élevées, activation d’un partenaire, déblocage à risque, export massif, publication de chiffres officiels, changement de compte de règlement et action distante critique sur TPE.

## UX/UI

Next.js App Router, React, TypeScript et Tailwind CSS. Le portail partage les tokens du système premium mais privilégie lisibilité, densité et rapidité. Framer Motion est limité aux transitions utiles. GSAP et 3D ne sont utilisés que sur l’accueil, la visualisation pédagogique ou des démonstrations, jamais dans les parcours critiques.

## Sécurité

MFA obligatoire pour rôles sensibles, sessions courtes et renouvelables, step-up authentication, CSP stricte, protection CSRF selon architecture, validation serveur, chiffrement, contrôle d’export, masquage, rate limiting, détection d’anomalies, journalisation et revue régulière des permissions.

## Critères d’acceptation

- Un utilisateur ne voit ni n’appelle une action non autorisée.
- Toute action critique produit une trace d’audit complète.
- La double validation empêche l’auteur d’approuver sa propre demande.
- Les tableaux supportent pagination serveur, filtres sauvegardés et export contrôlé.
- Les environnements sont impossibles à confondre visuellement.
- Les données sensibles sont masquées par défaut.
- Les écrans restent utilisables clavier et lecteur d’écran.
- Les erreurs partenaires sont corrélées sans exposer de secret.
- Les modifications CMS ont brouillon, prévisualisation, approbation et historique.