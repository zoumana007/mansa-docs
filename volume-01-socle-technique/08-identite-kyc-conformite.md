# Identité, KYC et conformité

## Objectif

Ce document définit le cycle de vérification d’identité commun aux applications Mansa. Il complète les contrats techniques de `packages/contracts/src/kyc.ts` et `packages/contracts/src/kyc-api.ts` dans `mansa-platform`.

## Principes

- Une vérification KYC est portée par un dossier versionné.
- Un dossier appartient à un utilisateur, un pays et un programme de conformité.
- Les documents réels ne sont jamais stockés dans les contrats ni dans les journaux applicatifs : seul un identifiant d’objet sécurisé est conservé.
- Les numéros de documents sont masqués avant exposition.
- Toute soumission, modification documentaire et décision est idempotente et auditée.
- Une décision administrative doit conserver le motif interne, l’auteur, la date et le niveau KYC résultant.
- Le demandeur ne reçoit jamais les commentaires internes de conformité.

## Niveaux

Les niveaux partagés sont :

- `UNVERIFIED` : identité non vérifiée ;
- `BASIC` : informations minimales vérifiées ;
- `STANDARD` : identité et justificatifs principaux vérifiés ;
- `ENHANCED` : vigilance renforcée ;
- `REJECTED` : vérification refusée ;
- `SUSPENDED` : niveau précédemment accordé mais suspendu.

Les limites de compte, paiement, transfert, carte et retrait doivent être calculées à partir du niveau effectif, du pays, du profil de risque et des règles configurées par l’administration.

## Cycle de vie du dossier

`DRAFT` → `SUBMITTED` → `IN_REVIEW` → `APPROVED`

Les variantes autorisées sont :

- `IN_REVIEW` → `ACTION_REQUIRED` lorsqu’une correction ou un document supplémentaire est nécessaire ;
- `ACTION_REQUIRED` → `SUBMITTED` après nouvelle soumission ;
- `IN_REVIEW` → `REJECTED` ;
- `DRAFT` ou `ACTION_REQUIRED` → `CANCELLED` selon les règles du programme.

Une transition invalide doit retourner une erreur métier explicite et ne produire aucun effet partiel.

## Contrat API

Les routes de référence sont :

| Opération | Méthode | Route |
|---|---|---|
| Créer un brouillon | `POST` | `/v1/kyc/cases` |
| Lister les dossiers | `GET` | `/v1/kyc/cases` |
| Lire un dossier | `GET` | `/v1/kyc/cases/:caseId` |
| Modifier le profil | `PATCH` | `/v1/kyc/cases/:caseId/profile` |
| Ajouter un document | `POST` | `/v1/kyc/cases/:caseId/documents` |
| Retirer un document | `DELETE` | `/v1/kyc/cases/:caseId/documents/:documentId` |
| Soumettre le dossier | `POST` | `/v1/kyc/cases/:caseId/submit` |
| Décider côté administration | `POST` | `/v1/admin/kyc/cases/:caseId/review` |

Les commandes de mutation transportent une clé d’idempotence. Les modifications d’un dossier utilisent aussi `expectedVersion` afin d’empêcher l’écrasement silencieux de changements concurrents.

## Autorisations minimales

- Un client peut créer, consulter et modifier uniquement son propre dossier tant qu’il est modifiable.
- Un agent support peut consulter le statut et les messages destinés au client, mais pas les commentaires internes ni les documents bruts.
- Un analyste conformité peut examiner les dossiers de son périmètre pays/programme.
- La décision finale peut exiger une seconde validation selon le niveau de risque.
- Un super administrateur ne peut pas contourner les règles de séparation des tâches sans procédure d’urgence auditée.

## Protection des données

- Chiffrement en transit et au repos obligatoire.
- Accès aux documents via URL temporaire ou flux authentifié à durée limitée.
- Analyse antivirus et contrôle du type réel de fichier avant acceptation.
- Rétention configurable selon le pays et le contrat partenaire.
- Journalisation sans donnée biométrique, numéro complet, image de pièce ou adresse complète.
- Export et suppression soumis aux obligations légales de conservation.

## Critères d’acceptation

1. Deux requêtes avec la même clé d’idempotence ne créent pas deux dossiers ni deux décisions.
2. Une version obsolète est rejetée avec une erreur de concurrence.
3. Un utilisateur ne peut jamais lire le dossier d’un autre utilisateur.
4. Un agent sans périmètre pays/programme ne peut pas décider un dossier.
5. Aucun secret, document brut ou numéro complet n’apparaît dans les logs.
6. Chaque transition produit un événement d’audit corrélé.
7. Les filtres de liste couvrent utilisateur, pays, programme, statut, niveau résultant et période.
8. La décision `APPROVE` exige un niveau résultant compatible avec le programme.
9. Une décision `REJECT` ou `REQUEST_ACTION` exige un code motif interne.
10. Les contrats TypeScript et la documentation conservent les mêmes routes, méthodes et statuts.

## Éléments restant à implémenter

- validation des transitions dans le domaine ;
- persistance du dossier et verrouillage optimiste ;
- stockage sécurisé et analyse des documents ;
- intégration fournisseur KYC derrière adaptateur ;
- écrans Client et Admin ;
- règles de limites par niveau ;
- événements, métriques, alertes et tableaux de suivi conformité.
