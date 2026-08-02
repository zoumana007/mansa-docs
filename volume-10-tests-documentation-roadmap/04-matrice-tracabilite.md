# Matrice de traçabilité des exigences

## Objectif

Cette matrice relie les exigences fonctionnelles et techniques aux composants du monorepo, aux validations attendues et aux preuves de recette. Elle doit être mise à jour à chaque ajout de module significatif.

## Convention d’identifiants

- `SOC-*` : socle technique et architecture.
- `IAM-*` : identité, authentification et autorisations.
- `KYC-*` : connaissance client et conformité.
- `LED-*` : comptes, soldes et grand livre.
- `PAY-*` : paiements, transferts et encaissements.
- `CRD-*` : cartes physiques et virtuelles.
- `MER-*` : commerçants et employés.
- `TPE-*` : terminal de paiement.
- `ADM-*` : administration et gouvernance.
- `ETA-*` : services publics et module État.
- `AI-*` : services IA contrôlés.
- `DAT-*` : données, reporting et exports.
- `OPS-*` : exploitation, sécurité et production.

## Matrice initiale

| ID | Exigence | Composant principal | Preuve attendue | Statut |
|---|---|---|---|---|
| SOC-001 | Monorepo installable avec une commande unique | racine, `pnpm-workspace.yaml` | installation CI réussie | À construire |
| SOC-002 | TypeScript strict pour les services compatibles | `packages/config`, applications TS | `pnpm typecheck` sans erreur | À construire |
| SOC-003 | Séparation Démo, Recette et Production | configuration, infrastructure | tests de configuration et revue | À construire |
| IAM-001 | Authentification sécurisée avec rotation des jetons | `apps/api-gateway` | tests unitaires et d’intégration | À construire |
| IAM-002 | RBAC/ABAC et moindre privilège | `packages/security`, API, admin | matrice de permissions testée | À construire |
| KYC-001 | Parcours KYC versionné et auditable | API, client, admin | scénario de recette complet | À construire |
| LED-001 | Montants stockés en unités mineures entières | `packages/domain`, base | tests de propriétés et revue du schéma | Partiel |
| LED-002 | Grand livre en partie double équilibré | API, workers, base | invariant débit = crédit | À construire |
| PAY-001 | Idempotence des opérations financières | API, contrats | rejeu sans double débit | Partiel |
| PAY-002 | Webhooks signés, rejouables et dédupliqués | API, workers | tests de signature et rejeu | À construire |
| CRD-001 | Gestion du cycle de vie des cartes | API, client, admin | émission simulée, blocage, remplacement | À construire |
| MER-001 | Gestion des commerces, points de vente et employés | API, applications commerçant | tests d’autorisations par établissement | À construire |
| TPE-001 | Paiement TPE avec adaptateur matériel isolé | `apps/tpe-android` | tests sur simulateur puis terminal certifié | À construire |
| ADM-001 | Toute action sensible est auditée | admin, API, observabilité | journal horodaté, acteur, motif, corrélation | À construire |
| ADM-002 | Double validation pour les changements critiques | admin, API | recette maker-checker | À construire |
| ETA-001 | Identification forte des agents publics | module État, sécurité | test d’habilitation et révocation | À construire |
| ETA-002 | Amende traçable avec reçu infalsifiable | module État, paiements | recette de bout en bout en environnement démo | À construire |
| AI-001 | Les réponses Jini sont limitées par les permissions | IA, sécurité, contrats | tests d’accès et de fuite de données | À construire |
| DAT-001 | Exports financiers reproductibles et rapprochables | données, workers | comparaison export / grand livre | À construire |
| OPS-001 | Aucun secret réel n’est versionné | tous les dépôts | scan automatique des secrets | Partiel |
| OPS-002 | Sauvegarde et restauration sont testées | infrastructure | compte rendu de restauration | À construire |
| OPS-003 | Logs, métriques et traces partagent un identifiant de corrélation | observabilité | scénario distribué vérifié | À construire |

## Règles de mise à jour

1. Une exigence ne passe à `Terminé` que si le code, les tests et la documentation sont présents.
2. Une exigence `Partiel` doit préciser la preuve existante et l’élément manquant dans le document du module concerné.
3. Toute nouvelle fonctionnalité critique reçoit un identifiant avant fusion.
4. Les noms de composants doivent correspondre exactement aux chemins de `mansa-platform`.
5. Les preuves de recette ne doivent contenir ni secret ni donnée personnelle réelle.

## Définition de terminé

Une ligne est considérée terminée lorsque :

- l’implémentation est fusionnée sur la branche principale ;
- les tests automatisés pertinents sont verts ;
- les risques connus sont documentés ;
- les critères d’acceptation sont vérifiés ;
- les procédures manuelles restantes sont explicitement identifiées.
