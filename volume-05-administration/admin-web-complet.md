# Portail Admin Web complet

## Mission

`apps/admin-web` est le centre de contrôle opérationnel, fonctionnel, conformité et contenu de Mansa. Il ne doit pas être confondu avec le site public ni le site professionnel. Il utilise Next.js App Router, TypeScript et le design system partagé, avec des animations discrètes qui ne ralentissent jamais les opérations critiques.

## Domaines fonctionnels

- Dashboard global et tableaux de bord par pays, partenaire, produit et environnement.
- Utilisateurs, comptes, profils, appareils, sessions et restrictions.
- Commerçants, organisations, établissements, employés, boutiques et mini-sites.
- TPE, parc matériel, affectations, état, version logicielle, configuration et incidents.
- Transactions, paiements, transferts, remboursements, annulations, rapprochements et exports.
- Wallets, ledger, soldes, écritures, réserves, limites et anomalies.
- Cartes physiques, virtuelles, temporaires et jetables, cycles de vie et contrôles.
- KYC/KYB, dossiers, pièces, statuts, revues, sanctions, listes de surveillance et preuves.
- Fraude, alertes, règles, scores, enquêtes, blocages et levées de blocage.
- Litiges, chargebacks, réclamations, preuves, décisions et communication.
- Support, tickets, conversations, SLA, macros, escalades et qualité.
- Notifications, modèles, campagnes, canaux, consentements et journaux d’envoi.
- CMS des sites public et professionnel.
- Administration de l’Annuaire/Hub, modération, catégories, visibilité et offres mises en avant.
- Tarification, commissions, taxes, promotions, plafonds et exceptions.
- Pays, devises, langues, fuseaux horaires et règles locales.
- Partenaires, banques, Mobile Money, processeurs cartes et adaptateurs.
- Module État : administrations, agents, amendes, taxes, bourses, cartes étudiantes et services publics.
- IA/Jini : configuration, sources, règles, évaluations, incidents, garde-fous et revue humaine.
- Analytics, rapports, exports, indicateurs réglementaires et qualité des données.
- Monitoring, santé des services, incidents, maintenance et communication de statut.
- Feature flags et configuration Démo, Recette, Production.
- Centre de conformité, journaux techniques et audit.

## Rôles et permissions

Le portail applique RBAC et ABAC. Les permissions sont atomiques : lecture, création, modification, approbation, export, blocage, remboursement, gestion de secret, changement de configuration et accès aux données sensibles.

Exemples de rôles : Super Admin, Admin pays, Opérations, Finance, Conformité, Risque/Fraude, Support, Partenariats, Contenu/CMS, État, Auditeur, Observabilité et Lecture seule. Les rôles ne donnent pas automatiquement accès à tous les pays ou organisations.

## Double validation

Une double validation indépendante est requise selon seuil et criticité pour :

- modification de commissions ou plafonds ;
- remboursement important ou écriture manuelle ;
- déblocage après alerte fraude ;
- modification de permissions privilégiées ;
- activation d’un partenaire en production ;
- changement de configuration financière ;
- export massif de données ;
- changement d’une règle État sensible.

Le demandeur ne peut pas être son propre approbateur.

## Audit

Chaque action sensible enregistre : acteur, rôle, organisation, pays, environnement, date, IP ou contexte réseau, appareil, cible, ancienne valeur, nouvelle valeur, justification, identifiant de corrélation, résultat et approbateur éventuel. L’interface ne permet jamais de modifier ou supprimer un audit.

## Secrets

Le portail ne montre pas les valeurs secrètes. Il manipule uniquement des références de secret, versions, métadonnées, dates de rotation et états de connexion. Toute révélation exceptionnelle doit passer par un système externe audité.

## UX

- Navigation globale avec recherche et commande rapide.
- Tables virtualisées, filtres persistants, vues enregistrées et exports autorisés.
- Fiches 360° utilisateurs, commerçants, TPE et transactions.
- État de chargement, vide, erreur et permission refusée explicites.
- Actions destructrices confirmées avec résumé de l’impact.
- Animations courtes et désactivables ; aucune scène 3D dans les tâches opérationnelles.

## API et événements

Le portail consomme des contrats versionnés. Toute mutation exige authentification renforcée, contrôle d’autorisation côté serveur, idempotency key lorsque nécessaire et journal d’audit. Les mises à jour longues produisent des jobs consultables plutôt que de bloquer la requête.

## Critères d’acceptation

- Un utilisateur ne voit que les modules, pays, organisations et champs autorisés.
- Une permission refusée côté interface reste refusée côté API.
- Une action à double validation ne peut pas être exécutée par le demandeur seul.
- Toute action sensible est retrouvable par identifiant de corrélation.
- Les exports respectent filtrage, masquage et journalisation.
- Les environnements sont visuellement distincts et les actions Production sont renforcées.
- Le CMS publie avec brouillon, prévisualisation, approbation et historique de versions.
- Les écrans principaux restent utilisables au clavier et avec reduced motion.