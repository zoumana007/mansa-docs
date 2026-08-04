# Identité, KYC et conformité

## 1. Objectif

Ce chapitre définit le parcours de connaissance client (KYC) commun aux applications Mansa. Il complète les contrats techniques du paquet `@mansa/contracts`, notamment `kyc.ts` et `kyc-api.ts`.

Le KYC ne doit jamais être réduit à un simple formulaire. Il s'agit d'un dossier versionné, auditable et soumis à des règles différentes selon le pays, le produit, le niveau de risque et le partenaire financier.

## 2. Niveaux KYC

Les niveaux normalisés sont :

- `UNVERIFIED` : compte créé mais identité non vérifiée ;
- `BASIC` : identité minimale vérifiée, limites réduites ;
- `STANDARD` : dossier standard approuvé ;
- `ENHANCED` : contrôles renforcés et justificatifs complémentaires ;
- `REJECTED` : dossier refusé ;
- `SUSPENDED` : niveau précédemment accordé mais temporairement suspendu.

Les limites, produits disponibles et contrôles complémentaires sont configurés par programme KYC. Aucune application cliente ne doit coder ces limites en dur.

## 3. Cycle de vie d'un dossier

Un dossier suit les statuts suivants :

1. `DRAFT` : profil créé mais incomplet ;
2. `SUBMITTED` : dossier transmis avec une clé d'idempotence ;
3. `IN_REVIEW` : contrôle automatique ou manuel en cours ;
4. `ACTION_REQUIRED` : information ou document complémentaire demandé ;
5. `APPROVED` : niveau KYC accordé ;
6. `REJECTED` : dossier refusé avec un motif interne ;
7. `CANCELLED` : dossier abandonné ou remplacé.

Les transitions doivent être contrôlées côté serveur. La modification concurrente d'un dossier repose sur son champ `version` et sur `expectedVersion` dans les commandes sensibles.

## 4. Données minimales

Le profil comprend au minimum :

- nom et prénom légaux ;
- date et pays de naissance ;
- nationalité ;
- pays de résidence ;
- adresse ;
- profession et source des fonds lorsque le programme l'exige.

Les pièces prises en charge sont : carte nationale d'identité, passeport, titre de séjour, permis de conduire, justificatif de domicile et document complémentaire.

Le numéro de document ne doit jamais être exposé intégralement dans les réponses courantes. Le contrat utilise un numéro masqué et un identifiant d'objet de stockage séparé.

## 5. API de référence

Le contrat partagé expose les routes suivantes :

| Action | Méthode | Route |
|---|---|---|
| Créer un brouillon | `POST` | `/v1/kyc/cases` |
| Lire un dossier | `GET` | `/v1/kyc/cases/:caseId` |
| Lister les dossiers | `GET` | `/v1/kyc/cases` |
| Joindre un document | `POST` | `/v1/kyc/cases/:caseId/documents` |
| Soumettre un dossier | `POST` | `/v1/kyc/cases/:caseId/submit` |
| Décider un dossier | `POST` | `/v1/admin/kyc/cases/:caseId/review` |

Les opérations d'ajout de document et de soumission exigent une clé d'idempotence. La décision administrative exige une permission dédiée, une authentification renforcée et un événement d'audit.

## 6. Sécurité et confidentialité

- Les fichiers KYC sont stockés dans un stockage objet privé, chiffré et séparé de la base transactionnelle.
- Les liens de consultation sont temporaires et réservés aux utilisateurs autorisés.
- Les journaux ne doivent contenir ni image, ni numéro complet, ni donnée biométrique.
- Les actions de consultation, téléchargement, décision et export sont auditées.
- Les données de démonstration ne doivent jamais réutiliser de vraies pièces d'identité.
- La suppression logique et la durée de conservation sont pilotées par la politique réglementaire du pays et du partenaire.

## 7. Autorisations minimales

- Le client peut créer, lire et compléter son propre dossier.
- Un agent support peut consulter l'état sans accéder automatiquement aux documents.
- Un analyste conformité peut examiner les dossiers de son périmètre.
- Un décideur conformité peut approuver ou refuser selon les règles de séparation des tâches.
- Un super administrateur ne doit pas contourner silencieusement les contrôles ; toute dérogation est explicitement journalisée.

## 8. Événements attendus

Le module doit publier au minimum :

- `kyc.case.created` ;
- `kyc.document.attached` ;
- `kyc.case.submitted` ;
- `kyc.case.action_required` ;
- `kyc.case.approved` ;
- `kyc.case.rejected` ;
- `kyc.level.changed`.

Chaque événement comprend un identifiant, un horodatage, le dossier, l'utilisateur, le pays, le programme, la corrélation et la version du schéma.

## 9. Critères d'acceptation

- Une soumission répétée avec la même clé d'idempotence ne crée pas une seconde décision.
- Une version obsolète est rejetée avec une erreur de conflit.
- Un utilisateur ne peut pas lire le dossier d'un autre utilisateur.
- Un analyste sans permission de décision ne peut ni approuver ni refuser.
- Une décision crée un événement d'audit et met à jour le niveau KYC de manière atomique.
- Les documents privés ne sont jamais inclus directement dans les événements métier.
- Le contrat TypeScript, les routes NestJS et la documentation OpenAPI utilisent les mêmes noms et statuts.

## 10. Éléments à valider avant production

Les seuils, documents acceptés, contrôles de sanctions, règles de bénéficiaire effectif, conservation, recours client et responsabilités entre Mansa et la banque partenaire doivent être validés juridiquement et contractuellement pour chaque pays.
