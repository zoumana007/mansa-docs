# Décisions d’architecture du socle Mansa

Ce document fixe les décisions qui doivent rester cohérentes entre la documentation et le dépôt `zoumana007/mansa-platform`.

## ADR-001 — Monorepo pnpm

**Statut :** accepté.

Le code est organisé dans un monorepo pnpm avec des applications dans `apps/` et des bibliothèques partagées dans `packages/`.

Conséquences :

- les dépendances communes sont centralisées ;
- chaque application conserve ses scripts `build`, `lint`, `test` et `typecheck` ;
- la validation racine exécute les contrôles sur tous les espaces de travail ;
- aucun paquet applicatif ne doit importer directement le code interne d’une autre application.

## ADR-002 — API modulaire NestJS

**Statut :** accepté.

`apps/api-gateway` constitue le point d’entrée HTTP principal. Les domaines métier sont séparés en modules et dépendent de contrats et primitives partagés plutôt que de détails d’infrastructure.

Ordre de dépendance attendu :

1. domaine ;
2. cas d’usage ;
3. adaptateurs ;
4. transport HTTP, événements et tâches.

## ADR-003 — Montants en unités mineures

**Statut :** accepté.

Tous les montants sont représentés par des entiers en unités mineures et accompagnés d’un code devise ISO 4217. Les nombres flottants sont interdits pour les calculs financiers.

Exemple : `125000 XOF` représente 125 000 francs CFA, car XOF ne possède pas de subdivision utilisée dans la plateforme.

## ADR-004 — Grand livre en partie double

**Statut :** accepté.

Toute opération financière validée produit des écritures équilibrées. Une transaction ne peut pas être considérée comme comptabilisée si la somme des débits et des crédits n’est pas nulle dans la devise concernée.

Les corrections se font par écritures compensatrices ; les écritures comptabilisées ne sont jamais modifiées ni supprimées.

## ADR-005 — Idempotence obligatoire

**Statut :** accepté.

Les paiements, transferts, remboursements, webhooks et traitements asynchrones acceptent une clé d’idempotence. Une même clé, pour le même acteur et la même opération, doit retourner le résultat initial sans créer de doublon.

## ADR-006 — Adaptateurs partenaires

**Statut :** accepté.

Les banques, opérateurs Mobile Money, réseaux de cartes, fournisseurs KYC et services publics sont intégrés derrière des interfaces stables. Le cœur métier ne dépend pas des SDK propriétaires.

Chaque adaptateur doit gérer :

- délais d’attente et reprises contrôlées ;
- corrélation et idempotence ;
- traduction des erreurs ;
- journalisation sans données sensibles ;
- mode simulation pour Démo et Recette.

## ADR-007 — Séparation des environnements

**Statut :** accepté.

Démo, Recette et Production utilisent des comptes partenaires, secrets, bases, files et destinations de journalisation séparés. Aucun secret réel n’est versionné.

## ADR-008 — Autorisation RBAC et ABAC

**Statut :** accepté.

Le RBAC définit les capacités générales d’un rôle. L’ABAC applique ensuite les restrictions de pays, organisation, établissement, point de vente, montant, risque et environnement.

Les opérations sensibles peuvent exiger une double validation et sont toujours auditées.

## ADR-009 — Événements métier versionnés

**Statut :** accepté.

Les traitements asynchrones reposent sur des événements versionnés. Un consommateur doit tolérer les champs additionnels et refuser explicitement une version incompatible.

Les événements contiennent au minimum : identifiant, type, version, date, corrélation, causalité, environnement et référence de l’entité métier.

## ADR-010 — Observabilité sans fuite de données

**Statut :** accepté.

Chaque requête et traitement reçoit un identifiant de corrélation. Les journaux sont structurés, mais excluent mots de passe, jetons, CVV, PAN complet, documents KYC et données personnelles non nécessaires.

## Contrôle de cohérence

Avant fusion d’un changement structurel :

- vérifier les chemins dans le registre produit du monorepo ;
- mettre à jour cette liste si une décision est remplacée ;
- ajouter une décision locale dans `mansa-platform/docs/adr/` lorsqu’elle concerne directement l’implémentation ;
- vérifier que `pnpm validate` reste la commande de validation de référence.
