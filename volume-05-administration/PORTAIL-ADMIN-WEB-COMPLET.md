# Portail Admin Web complet

## Mission

Le portail Admin Web est le centre de contrôle de Mansa. Il administre tous les produits, sans accès implicite : chaque action dépend des permissions, du pays, de l’entité, de l’environnement et du niveau de risque.

## Modules

- Tableau de bord global : volumes, revenus, utilisateurs actifs, incidents, fraude, disponibilité et alertes.
- Utilisateurs : identité, appareils, sessions, limites, comptes, wallets, cartes, historique et support.
- Commerçants : KYB, établissements, équipes, catalogues, commissions, mini-sites, TPE, règlements et risques.
- TPE : parc, affectation, état, version, connectivité, clés référencées, transactions, journaux, maintenance et blocage.
- Transactions et ledger : recherche, corrélation, statuts, écritures, rapprochement, remboursement, annulation et litige.
- Cartes : émission, statut, plafonds, tokenisation, cartes virtuelles/temporaires/jetables et incidents.
- KYC/KYB : files de revue, preuves, décisions, motifs, escalade, expiration et audit.
- Sécurité : RBAC, ABAC, sessions, appareils, politiques, approbations, anomalies et révocations.
- Fraude et conformité : alertes, scénarios, score, dossiers, sanctions, déclarations et centre de conformité.
- Support : tickets, conversations, SLA, macros, pièces jointes sûres, escalade et satisfaction.
- CMS : contenu du site public et du site professionnel, publications, prévisualisation, versions et rollback.
- Annuaire/Hub : catégories, profils, vérification, visibilité, abonnements, offres, avis, modération et statistiques.
- Configuration : feature flags, pays, devises, langues, commissions, limites, partenaires, banques et Mobile Money.
- Module État : administrations, agents, taxes, amendes, bourses, cartes étudiantes, paiements publics et rapprochement.
- IA/Jini : versions de prompts, outils autorisés, garde-fous, taux d’erreur, évaluations et désactivation d’urgence.
- Analytics : funnels, rétention, revenus, cohortes, adoption, exports et rapports programmés.
- Exploitation : disponibilité, métriques, traces, logs, files, webhooks, incidents, maintenance et état des services.
- Environnements : Démo, Recette et Production clairement séparés, avec bannière et droits distincts.

## Rôles et permissions

Combiner RBAC et ABAC. Les attributs incluent pays, organisation, périmètre métier, environnement, montant, risque, heure, appareil et niveau de confiance. Les rôles initiaux : Super Admin restreint, Admin pays, Conformité, Fraude, Support, Opérations, Finance, Partenaires, État, CMS, Analyste, Auditeur lecture seule et Exploitation.

Le Super Admin n’est pas un contournement universel : les opérations critiques exigent une justification, une authentification renforcée et parfois une deuxième personne.

## Double validation

Exiger une double validation pour les changements de tarification, plafonds globaux, règles de fraude, activation partenaire, modification d’écritures compensatrices, export massif, suppression logique sensible, déblocage à risque, changement de clé/référence de secret et passage en production.

## Audit

Chaque action sensible enregistre acteur, rôle, organisation, environnement, date, IP, appareil, corrélation, objet, ancienne valeur, nouvelle valeur, justification, approbateur et résultat. Les journaux d’audit ne sont jamais modifiables depuis le portail.

## UX premium adaptée à l’administration

L’interface est haut de gamme mais privilégie la lisibilité et la vitesse. Les animations restent discrètes : transitions de contexte, chargements progressifs, graphiques fluides et micro-interactions. Les scènes 3D cinématiques sont réservées aux sites marketing ; elles ne doivent pas gêner les tâches administratives. Le portail respecte `prefers-reduced-motion`, le clavier, ARIA, les contrastes et les grands volumes de données.

## Écrans minimum

Connexion renforcée, tableau de bord, recherche globale, liste/détail utilisateur, liste/détail commerçant, transaction/ledger, KYC/KYB, cartes, TPE, fraude, litiges, support, CMS, Annuaire, module État, IA, analytics, monitoring, incidents, paramètres, rôles, approbations et audit.

## API et sécurité

Les API Admin utilisent un espace de noms dédié, des DTO stricts, pagination, filtres contrôlés, idempotence pour commandes, rate limiting, jetons courts, MFA, réauthentification des actions critiques et journalisation corrélée. Aucun secret réel n’est affiché ; seulement un identifiant ou une référence vers le gestionnaire externe.

## Critères d’acceptation

1. Un agent ne voit que son périmètre autorisé.
2. Une action critique sans deuxième approbateur échoue sans effet.
3. Toute mutation sensible produit une entrée d’audit exploitable.
4. Les environnements sont visuellement et techniquement séparés.
5. Les tableaux restent utilisables au clavier et avec lecteur d’écran.
6. Une recherche globale ne divulgue aucune donnée hors périmètre.
7. Les exports massifs sont autorisés, tracés, limités et chiffrés.
8. Le CMS permet brouillon, prévisualisation, publication et rollback.
9. Une configuration peut être désactivée rapidement via feature flag.
10. Les actions financières ne modifient jamais directement le ledger sans commande métier contrôlée.