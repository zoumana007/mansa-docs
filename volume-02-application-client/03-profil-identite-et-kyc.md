# Profil, identité et vérification KYC

## 1. Objectif

Ce module gère l’identité déclarée et vérifiée de l’utilisateur, son profil public et privé, les justificatifs, les contrôles réglementaires et les droits fonctionnels qui dépendent du niveau de vérification.

Le parcours doit être progressif : l’utilisateur peut découvrir l’application et commencer son inscription sans fournir immédiatement toutes les pièces, mais aucune fonction réglementée ne doit être ouverte avant le niveau requis.

## 2. Principes

- Séparer les données de profil, les données d’identité et les preuves documentaires.
- Ne collecter que les informations nécessaires au produit, au pays et au niveau de risque.
- Chiffrer les données sensibles au repos et en transit.
- Journaliser tout accès administratif aux données KYC.
- Ne jamais modifier silencieusement une identité déjà vérifiée.
- Conserver la décision, la règle appliquée et la version du contrôle.
- Permettre une reprise claire après rejet, expiration ou suspension.

## 3. États du dossier

Le domaine utilise les états suivants :

- `not_started` : aucun parcours commencé ;
- `in_progress` : informations ou pièces en cours de collecte ;
- `pending_review` : dossier transmis pour contrôle automatique ou manuel ;
- `verified` : identité validée ;
- `rejected` : dossier refusé avec motif exploitable ;
- `expired` : vérification ou document arrivé à expiration ;
- `suspended` : vérification temporairement neutralisée après un signal de risque ou une décision réglementaire.

Les transitions doivent être contrôlées. Un dossier ne passe jamais directement de `not_started` à `verified`.

## 4. Données de profil

Le profil peut contenir :

- prénom et nom d’usage ;
- photo ou avatar ;
- identifiant Mansa ;
- langue ;
- pays et région ;
- préférences de notification ;
- profession ou secteur lorsque nécessaire ;
- adresse de résidence ;
- contacts vérifiés ;
- paramètres d’accessibilité.

Le profil public ne doit jamais exposer automatiquement la date de naissance, l’adresse, les documents, le numéro officiel ou les données de conformité.

## 5. Données d’identité

Selon le pays et le niveau demandé :

- nom légal ;
- prénoms légaux ;
- date et lieu de naissance ;
- nationalité ;
- sexe lorsque légalement nécessaire ;
- type et numéro de document ;
- autorité émettrice ;
- date d’émission et d’expiration ;
- adresse ;
- numéro fiscal ou identifiant national lorsque requis.

Les champs obligatoires sont configurés par pays, produit, partenaire et niveau de compte.

## 6. Pièces acceptées

Exemples configurables :

- carte nationale d’identité ;
- passeport ;
- titre de séjour ;
- permis de conduire lorsque accepté ;
- extrait ou acte administratif ;
- justificatif de domicile ;
- attestation professionnelle ;
- document fiscal ;
- certificat d’immatriculation d’entreprise.

Chaque type possède une politique : pays, durée de validité, faces requises, qualité minimale, contrôles et date d’expiration.

## 7. Parcours particulier

1. Présentation du motif de la collecte.
2. Consentement et information sur les données.
3. Saisie des informations personnelles.
4. Choix du document.
5. Capture recto-verso si nécessaire.
6. Contrôle de qualité en temps réel.
7. Lecture automatique et comparaison avec la saisie.
8. Selfie et preuve de vie selon le risque.
9. Vérification des listes et règles applicables.
10. Résumé avant envoi.
11. Passage en `pending_review`.
12. Décision et notification.

L’utilisateur doit pouvoir sauvegarder un brouillon sans envoyer un dossier incomplet.

## 8. Contrôles documentaires

Les contrôles peuvent inclure :

- format et cohérence du numéro ;
- document non expiré ;
- détection de recadrage ou retouche ;
- présence des éléments de sécurité visibles ;
- comparaison OCR avec les données saisies ;
- cohérence entre recto et verso ;
- détection de doublon ;
- comparaison du visage ;
- preuve de vie ;
- contrôle de majorité et d’éligibilité ;
- vérification auprès d’une source autorisée lorsqu’un contrat le permet.

Une décision automatique à faible confiance doit être envoyée en revue manuelle plutôt que rejetée définitivement.

## 9. Niveaux de vérification

Les niveaux sont configurables. Exemple :

### Niveau 0 — Découverte

- consultation limitée ;
- mode démonstration ;
- aucune transaction réelle.

### Niveau 1 — Contact vérifié

- téléphone ou e-mail vérifié ;
- faibles plafonds ;
- fonctionnalités limitées.

### Niveau 2 — Identité vérifiée

- document et identité validés ;
- portefeuille complet selon le pays ;
- cartes et transferts sous réserve des partenaires.

### Niveau 3 — Renforcé

- adresse, revenus, activité ou justificatifs supplémentaires ;
- plafonds supérieurs ;
- produits à risque ou professionnels.

Un niveau ne garantit pas automatiquement l’accès à tous les produits : les règles de risque et de conformité restent applicables.

## 10. Rejet

Le rejet doit comporter :

- une catégorie interne précise ;
- un message utilisateur compréhensible ;
- les éléments à corriger ;
- la possibilité ou non de recommencer ;
- le nombre de nouvelles tentatives ;
- la nécessité éventuelle d’une revue manuelle.

Les motifs sensibles liés aux contrôles réglementaires ne sont pas toujours communicables intégralement.

## 11. Expiration et renouvellement

Le système surveille :

- expiration du document ;
- durée maximale depuis la dernière vérification ;
- changement de données critiques ;
- évolution réglementaire ;
- niveau de risque.

Avant expiration, l’utilisateur reçoit des rappels. Après expiration, les droits sont réduits progressivement selon la réglementation et le risque, sans supprimer l’historique du compte.

## 12. Suspension

Une identité vérifiée peut être suspendue en cas de :

- document signalé ;
- incohérence grave ;
- fraude suspectée ;
- changement d’identité non confirmé ;
- demande d’une autorité compétente ;
- compte compromis ;
- contrôle périodique incomplet.

La suspension est distincte du blocage financier. Chaque restriction doit être explicitement appliquée par le moteur de règles.

## 13. Modification d’une identité vérifiée

Les changements de nom légal, date de naissance, nationalité ou document doivent :

- exiger une réauthentification ;
- créer une demande de modification ;
- conserver l’ancienne valeur dans l’audit ;
- demander une nouvelle preuve ;
- passer en revue ;
- notifier l’utilisateur ;
- réévaluer les produits et plafonds.

Les corrections typographiques simples peuvent suivre une procédure allégée, mais restent tracées.

## 14. Entreprises et commerçants

Le KYC personne physique est complété par un KYB comprenant :

- raison sociale ;
- forme juridique ;
- immatriculation ;
- adresse ;
- activité ;
- dirigeants ;
- bénéficiaires effectifs ;
- représentants autorisés ;
- compte bancaire de règlement ;
- licences éventuelles ;
- preuves fiscales.

Les rôles de propriétaire, administrateur et employé doivent être distincts.

## 15. Administration

Les agents autorisés peuvent :

- consulter une file de dossiers ;
- filtrer par pays, risque, ancienneté et statut ;
- voir les contrôles et leurs scores ;
- demander une pièce supplémentaire ;
- approuver, rejeter, suspendre ou rouvrir ;
- ajouter une note interne ;
- escalader vers conformité ;
- consulter l’historique complet.

Les agents ne peuvent pas télécharger librement les documents. Les droits, motifs d’accès et durées de consultation sont contrôlés.

## 16. API et événements

Commandes principales :

- démarrer un dossier ;
- enregistrer un brouillon ;
- ajouter ou remplacer une pièce ;
- soumettre ;
- demander un complément ;
- décider ;
- suspendre ;
- renouveler ;
- clôturer.

Événements :

- `kyc.started` ;
- `kyc.submitted` ;
- `kyc.additional_information_requested` ;
- `kyc.verified` ;
- `kyc.rejected` ;
- `kyc.expired` ;
- `kyc.suspended` ;
- `kyc.renewal_started`.

Chaque commande sensible doit être idempotente et chaque événement doit contenir un identifiant de corrélation.

## 17. Cohérence avec le code

Le paquet `packages/domain` de `mansa-platform` expose :

- `KycStatus` pour les états autorisés ;
- `KycState` pour contrôler les transitions ;
- `InvalidKycTransitionError` pour refuser les parcours incohérents ;
- `isKycStatus` pour valider les valeurs reçues aux frontières du système.

La persistance, les fournisseurs KYC, les API et l’interface doivent réutiliser ce contrat au lieu de redéfinir leurs propres statuts.

## 18. Critères d’acceptation

- Un utilisateur peut commencer, interrompre et reprendre son dossier.
- Une transition invalide est refusée par le domaine.
- Un dossier soumis devient `pending_review`.
- Une approbation produit `verified` et déclenche la réévaluation des droits.
- Un rejet permet une nouvelle tentative uniquement si la politique l’autorise.
- Une expiration ou suspension retire les droits concernés sans effacer l’historique.
- Toute consultation ou décision administrative est auditée.
- Aucun secret, document réel ou donnée personnelle de production n’est stocké dans Git.
