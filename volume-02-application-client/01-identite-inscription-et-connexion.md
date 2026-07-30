# Volume 2 — Identité, inscription et connexion

## 1. Objectif

L’application Client doit permettre de créer et sécuriser un compte Mansa sans mélanger l’identité civile, les moyens de connexion, les appareils et les autorisations. Le parcours reste utilisable sur réseau instable et toutes les étapes sensibles sont auditables.

## 2. États du compte

- `PENDING_VERIFICATION` : compte créé, coordonnées non vérifiées.
- `ACTIVE` : accès normal selon le niveau KYC.
- `LOCKED` : blocage de sécurité temporaire.
- `SUSPENDED` : suspension administrative ou conformité.
- `CLOSED` : clôture définitive, avec conservation réglementaire des traces.

Une transition d’état exige un motif, un acteur, un horodatage et une entrée d’audit.

## 3. Identifiants de connexion

Le téléphone est l’identifiant principal au lancement. L’e-mail peut être ajouté et vérifié. Les numéros sont normalisés au format E.164 avant stockage et comparaison. Un identifiant vérifié ne peut appartenir qu’à un seul compte actif dans un même périmètre pays.

Les mots de passe ne sont jamais stockés en clair. Le backend stocke uniquement un condensat produit par un algorithme adapté aux mots de passe et configurable. Les codes OTP sont courts, à usage unique, expirent rapidement et ne sont jamais journalisés.

## 4. Parcours d’inscription

1. Sélection du pays et acceptation des textes applicables.
2. Saisie et normalisation du numéro de téléphone.
3. Création d’un défi OTP avec limitation de débit.
4. Vérification du code.
5. Définition du secret de connexion ou activation d’une méthode forte.
6. Création du profil minimal et du portefeuille initial selon les règles du pays.
7. Enregistrement de l’appareil et émission d’une session.
8. Démarrage du parcours KYC adapté aux limites demandées.

Toutes les opérations de création utilisent une clé d’idempotence afin d’éviter les comptes ou portefeuilles en double.

## 5. Connexion et sessions

- Jeton d’accès de courte durée.
- Jeton de renouvellement rotatif, révocable et associé à une session serveur.
- Empreinte d’appareil pseudonymisée.
- Révocation à la déconnexion, au changement de secret, au verrouillage du compte ou sur décision administrative.
- Liste des appareils et sessions visibles par le client.
- Authentification renforcée avant ajout de bénéficiaire, changement de téléphone, paiement important ou consultation d’informations sensibles.

## 6. Protection contre les abus

- Limitation par compte, téléphone, appareil et adresse réseau.
- Temporisation progressive après échecs.
- Détection des tentatives de réutilisation d’un OTP ou d’un jeton de renouvellement.
- Blocage automatique configurable, sans révéler si un compte existe.
- Notifications de nouvelle connexion et de changement sensible.
- Procédure de récupération séparée de la connexion normale.

## 7. Autorisations

L’application Client utilise des permissions métier explicites. Un rôle ne suffit pas à lui seul : l’état du compte, le niveau KYC, les limites, le pays, le risque de la transaction et les blocages actifs sont aussi évalués.

## 8. Données minimales

- identifiant utilisateur interne non signifiant ;
- statut du compte ;
- pays de rattachement ;
- téléphone et e-mail normalisés avec état de vérification ;
- préférences de langue ;
- dates de création, mise à jour et dernière activité ;
- sessions et appareils ;
- consentements versionnés ;
- références KYC, sans dupliquer inutilement les documents.

## 9. Critères d’acceptation

- Un OTP expiré, déjà consommé ou dépassant le nombre d’essais est refusé.
- Une session révoquée ne peut pas être renouvelée.
- La rotation d’un jeton invalide immédiatement sa version précédente.
- Aucun message public ne permet de confirmer l’existence d’un compte.
- Les changements d’état, de téléphone, de secret et de permissions sont audités.
- Les contrats utilisés par l’API correspondent aux types partagés du dépôt `mansa-platform`.
