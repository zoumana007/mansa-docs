# Identité, KYC et conformité du client

## 1. Objectif

Le parcours KYC permet d’identifier un client, de collecter uniquement les informations nécessaires, de soumettre les justificatifs attendus et de produire un niveau de vérification exploitable par les règles de limites, de paiement et de risque.

Le parcours ne doit jamais simuler une validation réglementaire réelle : les programmes, documents acceptés, durées de conservation et décisions doivent être validés avec la banque partenaire, les autorités compétentes et les conseils juridiques de chaque pays.

## 2. Niveaux de vérification

Les niveaux partagés sont :

- `UNVERIFIED` : compte créé mais identité non validée ;
- `BASIC` : vérification minimale autorisant uniquement les usages configurés ;
- `STANDARD` : vérification standard permettant les plafonds ordinaires ;
- `ENHANCED` : vigilance renforcée et plafonds ou produits spécifiques ;
- `REJECTED` : dossier refusé ;
- `SUSPENDED` : niveau précédemment accordé temporairement suspendu.

Un niveau ne donne aucun droit par lui-même. Les autorisations effectives proviennent des politiques administrables par pays, programme, produit et profil de risque.

## 3. Cycle de vie d’un dossier

Le dossier suit les statuts `DRAFT`, `SUBMITTED`, `IN_REVIEW`, `ACTION_REQUIRED`, `APPROVED`, `REJECTED` et `CANCELLED`.

Règles obligatoires :

1. Un brouillon appartient à un seul utilisateur et à un seul programme KYC.
2. Toute modification incrémente `version` afin d’empêcher les écrasements concurrents.
3. La soumission exige une clé d’idempotence, la version attendue et la liste exacte des documents à considérer.
4. Après soumission, le client ne modifie pas silencieusement les données examinées.
5. Une demande complémentaire repasse le dossier à `ACTION_REQUIRED` et expose au client un message traduisible, sans commentaire interne.
6. Une décision administrative conserve le décideur, le motif interne, l’horodatage et l’événement d’audit.
7. Un dossier approuvé ne peut être altéré ; une nouvelle vérification crée une nouvelle version ou un nouveau dossier selon la politique du programme.

## 4. Données collectées

Le profil couvre l’état civil, la date et le pays de naissance, la nationalité, la résidence, l’adresse, la localité et, lorsque la politique le requiert, l’activité et la source des fonds.

Les documents autorisés par le contrat initial sont : carte nationale d’identité, passeport, titre de séjour, permis de conduire, justificatif de domicile et document complémentaire.

Le numéro complet d’un document ne doit jamais apparaître dans les journaux, notifications ou réponses non privilégiées. Les fichiers sont stockés dans un service d’objets sécurisé ; le contrat ne transporte qu’un `storageObjectId` et une valeur masquée éventuelle.

## 5. Contrat API

Le dépôt plateforme expose `@mansa/contracts/kyc-api` et les routes suivantes :

| Opération | Méthode | Route |
|---|---|---|
| Créer un brouillon | `POST` | `/v1/kyc/cases` |
| Consulter un dossier | `GET` | `/v1/kyc/cases/:caseId` |
| Rechercher les dossiers | `GET` | `/v1/kyc/cases` |
| Ajouter un document | `POST` | `/v1/kyc/cases/:caseId/documents` |
| Retirer un document | `DELETE` | `/v1/kyc/cases/:caseId/documents/:documentId` |
| Soumettre le dossier | `POST` | `/v1/kyc/cases/:caseId/submit` |
| Décider administrativement | `POST` | `/v1/admin/kyc/cases/:caseId/review` |

La recherche administrative est paginée et filtrable par utilisateur, pays, programme, statut et période de création.

## 6. Sécurité et confidentialité

- Chiffrement en transit et au repos obligatoire.
- Autorisation objet par objet : un client ne consulte que ses dossiers.
- Accès administratif limité aux rôles conformité habilités.
- Aucune pièce KYC réelle dans les environnements Démo ou les dépôts Git.
- Détection antivirus et validation du type réel de fichier avant stockage.
- URL de téléchargement courte durée, non réutilisable et auditée.
- Masquage systématique des identifiants et numéros de document.
- Politique de conservation et suppression configurable selon le pays et le motif légal.
- Double validation requise pour les décisions à risque élevé lorsque la politique le prévoit.

## 7. Événements attendus

Le backend publie au minimum des événements corrélés pour la création, la soumission, l’entrée en revue, la demande complémentaire, l’approbation, le rejet, l’annulation et le changement de niveau KYC.

Les consommateurs utilisent ces événements pour recalculer les limites, activer ou bloquer des produits, produire les notifications et alimenter l’audit. Aucun événement ne contient le fichier du document ni ses données non masquées.

## 8. Critères d’acceptation

- Deux soumissions avec la même clé d’idempotence produisent le même résultat.
- Une version obsolète renvoie un conflit sans écraser la version courante.
- Un utilisateur ne peut ni lire ni modifier le dossier d’un autre utilisateur.
- Un agent sans permission conformité ne peut prendre aucune décision.
- Une approbation sans niveau résultant valide est refusée.
- Une décision, un ajout ou un retrait de document produit un audit corrélable.
- Les réponses publiques ne révèlent ni commentaire interne ni numéro complet de document.
- Les filtres et la pagination du back-office donnent des résultats déterministes.

## 9. Correspondance documentation-code

Les types métier sont définis dans `packages/contracts/src/kyc.ts`. Le contrat transport est défini dans `packages/contracts/src/kyc-api.ts` et exporté par le sous-chemin `@mansa/contracts/kyc-api`.
