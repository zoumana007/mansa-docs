# Volume 10 — Stratégie de tests et critères de sortie

## 1. Objectif

La stratégie de tests de Mansa vise d’abord l’intégrité financière, la sécurité, la compatibilité des contrats et la capacité de reprise. Une fonctionnalité visible mais non vérifiable n’est pas considérée comme terminée.

## 2. Pyramide de tests

### Tests unitaires

Ils couvrent les primitives monétaires, règles de frais, limites, transitions d’état, droits, validations, idempotence et décisions métier. Ils restent rapides et indépendants des services externes.

### Tests d’intégration

Ils vérifient les dépôts, transactions PostgreSQL, migrations Prisma, files, stockage et adaptateurs simulés. Les scénarios financiers contrôlent simultanément résultat métier, écritures du grand livre, audit et événement publié.

### Tests de contrat

Chaque API, événement et adaptateur possède un schéma versionné. Les producteurs et consommateurs sont testés contre les mêmes exemples. Une rupture incompatible impose une nouvelle version et un plan de migration.

### Tests de parcours

Ils couvrent les parcours critiques : inscription, KYC, alimentation, transfert, paiement commerçant, remboursement, carte, vente TPE, administration, service public et support.

### Tests non fonctionnels

- sécurité statique et dépendances ;
- recherche de secrets ;
- charge et endurance ;
- concurrence et idempotence ;
- résilience aux délais, doublons et messages désordonnés ;
- restauration de sauvegarde ;
- accessibilité et compatibilité des interfaces.

## 3. Scénarios financiers obligatoires

- Deux requêtes portant la même clé d’idempotence ne créent qu’une opération.
- Une opération échouée ne laisse aucune écriture partielle.
- Toute opération finalisée est équilibrée dans le grand livre.
- Un remboursement ne dépasse jamais le montant remboursable.
- Une compensation crée de nouvelles écritures et ne modifie pas silencieusement l’historique.
- Les devises différentes ne sont jamais additionnées sans conversion explicite.
- Les arrondis suivent une règle unique et testée en unités mineures.
- Les reprises après délai partenaire ne produisent pas de double débit.

## 4. Données de test

Les données sont synthétiques. Aucun export de production, document KYC réel, numéro de carte complet, jeton ou secret ne doit être copié dans les fixtures, captures, journaux CI ou rapports.

Les jeux de données sont reproductibles, versionnés et séparés par environnement. Les tests ne dépendent pas de l’ordre d’exécution.

## 5. Pipeline minimal

À chaque pull request et push sur `main` :

1. installation reproductible ;
2. validation et génération Prisma ;
3. vérification du formatage ;
4. lint ;
5. vérification TypeScript stricte ;
6. tests ;
7. build ;
8. contrôles de sécurité configurés.

Les échecs bloquent la fusion. Les exceptions sont rares, datées, justifiées et approuvées.

## 6. Critères de sortie d’un module

Un module est terminé lorsque :

- ses exigences et états sont documentés ;
- ses contrats sont versionnés ;
- ses cas nominaux, erreurs et reprises sont testés ;
- ses permissions et audits sont vérifiés ;
- ses métriques, alertes et runbooks existent ;
- ses migrations sont compatibles et réversibles au niveau applicatif ;
- aucune donnée sensible n’apparaît dans les logs ;
- la CI est verte sur un clone propre ;
- la documentation correspond aux chemins et commandes réels.

## 7. Critères de préparation à la production

- Audit de sécurité indépendant clos ou risques formellement acceptés.
- Tests de charge représentatifs terminés.
- Restauration et reprise après sinistre exercées.
- Rapprochement financier validé avec chaque partenaire.
- Procédures d’incident, support et astreinte opérationnelles.
- Conformité juridique et réglementaire confirmée.
- Déploiement progressif et retour arrière testés.
- Responsables métier et techniques ayant donné leur accord.

## 8. Preuves conservées

Les rapports CI, versions d’artefacts, résultats de tests, approbations, migrations, audits et exercices de reprise sont conservés selon une politique définie. Ils doivent permettre de relier une version en production au code, aux contrôles et aux décisions correspondants.
