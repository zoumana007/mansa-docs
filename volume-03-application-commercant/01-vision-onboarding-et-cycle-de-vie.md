# Application Commerçant — Vision, onboarding et cycle de vie

## 1. Objectif

L’application Mansa Commerçant permet à une entreprise, un indépendant, une association autorisée ou un point de vente d’encaisser, rembourser, gérer ses employés, suivre ses ventes et administrer ses moyens de paiement. Elle partage le même cœur financier que les applications Client et TPE, sans dupliquer les règles de transaction.

## 2. Séparation des responsabilités

- **Profil commerçant** : identité légale, enseigne, pays, devise de règlement et état d’activation.
- **Point de vente** : emplacement physique ou virtuel, horaires, coordonnées et paramètres locaux.
- **Employé** : utilisateur rattaché à un ou plusieurs points de vente avec permissions limitées.
- **Terminal** : appareil TPE, téléphone ou session web autorisée à initier un encaissement.
- **Compte de règlement** : destination bancaire, portefeuille ou compte partenaire validé.

Une personne peut posséder plusieurs profils commerçants. Un profil commerçant peut exploiter plusieurs points de vente et plusieurs terminaux.

## 3. Données minimales du profil

Le domaine technique expose `MerchantProfile` dans `packages/domain/src/merchant-profile.ts` avec les champs suivants :

- `id` : identifiant interne immuable ;
- `ownerId` : propriétaire principal ;
- `legalName` : raison sociale ou nom légal ;
- `displayName` : enseigne affichée aux clients ;
- `countryCode` : code pays ISO 3166-1 alpha-2 ;
- `settlementCurrency` : devise ISO 4217 ;
- `status` : état courant ;
- `statusReason` : motif de rejet, suspension ou fermeture ;
- `createdAt` : date de création.

Les numéros fiscaux, documents, coordonnées bancaires et données KYC sensibles sont conservés dans des composants dédiés et ne doivent jamais apparaître dans les journaux applicatifs.

## 4. Cycle de vie

| État | Signification | Encaissement autorisé |
|---|---|---:|
| `draft` | Dossier créé mais non soumis | Non |
| `pending_review` | Dossier en contrôle | Non |
| `active` | Commerçant approuvé | Oui |
| `suspended` | Blocage temporaire motivé | Non |
| `rejected` | Dossier refusé | Non |
| `closed` | Relation terminée | Non |

Transitions autorisées :

1. `draft` → `pending_review` lors de la soumission.
2. `pending_review` → `active` après approbation.
3. `pending_review` → `rejected` avec motif obligatoire.
4. `active` → `suspended` avec motif obligatoire.
5. `suspended` → `active` après levée du blocage.
6. Tout état non final, sauf `rejected`, peut être fermé avec motif.

Aucune transition directe ne doit contourner la revue de conformité. La capacité d’accepter des paiements est vraie uniquement dans l’état `active`.

## 5. Parcours d’onboarding

1. Création du profil et choix du pays.
2. Saisie de l’identité légale et de l’enseigne.
3. Sélection du secteur d’activité et du type d’organisation.
4. Ajout des bénéficiaires effectifs et représentants.
5. Dépôt des documents requis selon le pays et le niveau de risque.
6. Configuration du compte de règlement.
7. Acceptation des conditions et consentements.
8. Soumission du dossier.
9. Contrôles automatiques puis revue humaine si nécessaire.
10. Activation, rejet motivé ou demande de complément.

Chaque étape doit être enregistrable en brouillon et reprise sur un autre appareil. Les pièces expirées ou insuffisantes déclenchent une demande de complément sans effacer le dossier.

## 6. Permissions initiales

- **Propriétaire** : toutes les opérations, bénéficiaires, règlements et fermeture.
- **Administrateur commerce** : points de vente, employés, catalogue et rapports.
- **Caissier** : encaissement, reçu et consultation limitée de son service.
- **Responsable** : encaissement, remboursement plafonné et clôture de caisse.
- **Comptable** : rapports, factures, exports et rapprochement, sans encaissement.

Les autorisations sont évaluées côté serveur. Masquer un bouton dans l’interface ne constitue jamais un contrôle de sécurité.

## 7. Règles d’acceptation

- Un profil incomplet reste en `draft`.
- Un profil non actif ne peut ni générer une intention de paiement commerçant exploitable ni recevoir un règlement.
- Un rejet, une suspension et une fermeture exigent un motif non vide et une trace d’audit.
- La réactivation efface le motif opérationnel courant sans supprimer l’historique d’audit.
- Les codes pays et devise sont normalisés en majuscules et validés.
- Les réponses API ne renvoient aucun document KYC ni secret de règlement par défaut.
- Toute modification du propriétaire, du pays ou du compte de règlement déclenche une nouvelle évaluation de risque.

## 8. Cohérence avec le code

Le premier agrégat de domaine est exporté par `packages/domain/src/index.ts` et couvert par `packages/domain/test/merchant-profile.test.mjs`. Les prochains lots doivent ajouter les points de vente, les employés et permissions, les comptes de règlement, les remboursements et la clôture de caisse sans déplacer les règles de cycle de vie dans les interfaces.
