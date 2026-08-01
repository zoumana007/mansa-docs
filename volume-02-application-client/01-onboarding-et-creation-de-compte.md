# Onboarding et création de compte — Mansa Client

## 1. Objectif

L’onboarding doit permettre à une personne de :

- comprendre rapidement ce qu’est Mansa ;
- créer un compte sans confusion ;
- sécuriser son accès ;
- choisir son identifiant Mansa ;
- commencer avec un niveau KYC progressif ;
- découvrir les fonctions principales ;
- utiliser l’application même avec une connexion faible.

L’inscription doit être simple, mais jamais au détriment de la sécurité ou des obligations réglementaires.

## 2. Principes généraux

### 2.1 Simplicité

L’utilisateur ne doit pas remplir un long formulaire unique. Le parcours doit être découpé en petites étapes claires.

### 2.2 Progressivité

Les informations non indispensables peuvent être demandées plus tard.

Exemples :

- téléphone : obligatoire immédiatement ;
- nom et date de naissance : selon le niveau de compte ;
- pièce d’identité : au moment du KYC ;
- profession ou revenus : uniquement si nécessaire.

### 2.3 Transparence

Avant toute demande, l’application doit expliquer :

- pourquoi l’information est demandée ;
- comment elle sera utilisée ;
- si elle est obligatoire ;
- ce qui reste inaccessible sans cette information.

### 2.4 Reprise automatique

Si l’utilisateur quitte le parcours, il doit pouvoir reprendre à la dernière étape validée.

Aucun mot de passe, PIN, OTP ou secret ne doit être conservé en clair.

### 2.5 Adaptation

Le parcours peut varier selon :

- le pays ;
- l’âge ;
- la réglementation ;
- le partenaire bancaire ;
- le type de compte ;
- le niveau KYC ;
- l’appareil ;
- la qualité de la connexion.

### 2.6 Accessibilité

Chaque écran doit fonctionner avec :

- lecteur d’écran ;
- taille de texte agrandie ;
- contraste renforcé ;
- réduction des animations ;
- navigation simple ;
- messages d’erreur compréhensibles.

## 3. Premier lancement

### 3.1 Splash Screen

Le premier écran affiche :

- logo Mansa ;
- animation légère ;
- version de l’application ;
- état de chargement.

L’animation doit être courte, avec une durée cible comprise entre 0,8 et 2 secondes.

Elle ne doit jamais masquer un chargement long. Si le chargement dépasse deux secondes, un texte doit apparaître :

> Préparation de Mansa…

### 3.2 Vérifications silencieuses

Pendant le chargement, l’application vérifie :

- version minimale autorisée ;
- maintenance ;
- disponibilité du backend ;
- pays détecté ;
- langue du téléphone ;
- compatibilité de l’appareil ;
- intégrité minimale de l’application ;
- présence d’une session existante ;
- état de la connexion.

### 3.3 Version obsolète

Si la version est trop ancienne :

- bloquer l’accès uniquement si la mise à jour est obligatoire ;
- afficher la raison ;
- proposer le bouton de mise à jour ;
- conserver les données locales sécurisées ;
- ne pas afficher une erreur technique incompréhensible.

### 3.4 Maintenance

En cas de maintenance :

- afficher le service concerné ;
- indiquer si l’application entière ou seulement certaines fonctions sont indisponibles ;
- afficher une estimation uniquement si elle est réellement connue ;
- permettre l’accès aux fonctions encore disponibles.

## 4. Choix du pays et de la langue

### 4.1 Détection automatique

L’application peut proposer un pays à partir de :

- carte SIM ;
- langue de l’appareil ;
- région du système ;
- choix précédent.

Elle ne doit pas utiliser la géolocalisation précise sans consentement.

### 4.2 Choix manuel

L’utilisateur peut choisir son pays dans une liste.

Chaque pays affiche :

- nom ;
- indicatif téléphonique ;
- devise principale ;
- disponibilité de Mansa ;
- fonctions actuellement disponibles.

### 4.3 Pays non disponible

Si Mansa n’est pas encore lancé dans ce pays :

- proposer une liste d’attente ;
- enregistrer l’e-mail ou le téléphone avec consentement ;
- ne pas permettre la création d’un faux compte actif ;
- informer clairement l’utilisateur.

### 4.4 Langue

La langue doit être modifiable immédiatement et plus tard dans les paramètres.

Langues initiales possibles :

- français ;
- anglais ;
- bambara ;
- autres langues activées par l’administration.

## 5. Écrans de bienvenue

Le nombre d’écrans doit rester limité : trois à cinq écrans maximum.

### 5.1 Écran 1 — Argent

Message :

> Gérez votre argent simplement.

Présenter :

- solde ;
- transferts ;
- paiements ;
- cartes.

### 5.2 Écran 2 — Mansa Connect

Message :

> Discutez, envoyez et demandez de l’argent dans la même conversation.

Présenter :

- messagerie ;
- demandes d’argent ;
- identifiant unique ;
- partage d’addition.

### 5.3 Écran 3 — Sécurité

Message :

> Votre sécurité reste sous votre contrôle.

Présenter :

- biométrie ;
- blocage de carte ;
- alertes ;
- appareils connectés.

### 5.4 Écran 4 — Jini

Message :

> Jini vous aide à comprendre et organiser votre argent.

Présenter :

- analyse ;
- rappels ;
- recherche ;
- recommandations.

### 5.5 Actions

Boutons :

- Créer un compte ;
- Se connecter ;
- Découvrir Mansa ;
- Passer la présentation.

## 6. Mode Découverte Interactif

### 6.1 Principe

Le mode Découverte est un environnement fictif totalement séparé des vraies données.

Il permet de tester l’application sans argent réel.

### 6.2 Données fictives

L’environnement peut contenir :

- faux solde ;
- fausse carte ;
- faux contact ;
- fausse conversation ;
- faux QR ;
- faux TPE ;
- faux reçu ;
- faux budget ;
- faux coffre.

### 6.3 Scénarios guidés

L’utilisateur peut tester :

- envoyer de l’argent ;
- demander de l’argent ;
- scanner un QR ;
- bloquer une carte ;
- consulter un reçu ;
- parler à Jini ;
- partager une addition ;
- créer un objectif.

### 6.4 Séparation stricte

Le mode Découverte doit utiliser :

- données fictives identifiées ;
- environnement séparé ;
- aucune transaction réelle ;
- aucune connexion à un partenaire de paiement réel ;
- visuels marqués « Démonstration ».

### 6.5 Sortie du mode Découverte

L’utilisateur peut quitter à tout moment.

Avant de revenir au compte réel, l’application affiche clairement :

> Vous quittez le mode Démonstration. Les prochaines opérations pourront utiliser de l’argent réel.

## 7. Création du compte

### 7.1 Choix du type de compte

Pour l’application Client, le parcours principal est le compte particulier.

Des variantes peuvent être proposées :

- compte jeune ;
- compte étudiant ;
- compte famille ;
- compte partenaire spécial.

Le compte commerçant complet doit être créé dans Mansa Commerce ou via un parcours professionnel dédié.

### 7.2 Numéro de téléphone

L’utilisateur saisit :

- pays ;
- indicatif ;
- numéro.

Validations :

- format ;
- longueur ;
- numéro déjà utilisé ;
- numéro interdit ;
- fréquence des tentatives ;
- cohérence avec le pays.

### 7.3 E-mail

L’e-mail peut être obligatoire, recommandé ou optionnel selon le pays et les règles de compte.

Il sert notamment à :

- récupération ;
- alertes ;
- reçus ;
- sécurité ;
- communication légale.

### 7.4 Consentements

Avant la création, présenter séparément :

- conditions générales ;
- politique de confidentialité ;
- communication commerciale facultative ;
- utilisation des contacts facultative ;
- analytics facultatifs lorsque la loi l’exige.

Les consentements obligatoires et facultatifs ne doivent pas être mélangés.

## 8. Vérification OTP

### 8.1 Canaux

Canaux possibles :

- SMS ;
- WhatsApp si autorisé ;
- e-mail ;
- appel vocal ;
- partenaire d’identité.

### 8.2 Règles

- code temporaire ;
- durée limitée ;
- nombre d’essais limité ;
- délai entre les renvois ;
- blocage temporaire en cas d’abus ;
- aucun OTP enregistré en clair.

### 8.3 Expérience

L’application doit :

- détecter automatiquement le code lorsque le système l’autorise ;
- afficher le délai restant ;
- permettre de corriger le numéro ;
- proposer un canal alternatif ;
- expliquer les erreurs.

### 8.4 Erreurs

Cas à gérer :

- code expiré ;
- code incorrect ;
- trop de tentatives ;
- SMS non reçu ;
- numéro déjà lié à un compte ;
- service OTP indisponible.

## 9. Création de l’identité Mansa

### 9.1 Identifiant unique

Chaque utilisateur choisit un identifiant unique, par exemple :

- `@zoumana` ;
- `@camara007` ;
- `@mansa-zou`.

### 9.2 Règles

L’identifiant doit :

- être unique ;
- être normalisé ;
- respecter une longueur minimale et maximale ;
- ne pas contenir de caractères interdits ;
- ne pas usurper une marque ou une institution ;
- ne pas être offensant ;
- ne pas être trompeur.

### 9.3 Vérification en temps réel

L’application affiche :

- Disponible ;
- Indisponible ;
- Réservé ;
- À vérifier ;
- Suggestions.

### 9.4 Modification

L’identifiant peut être modifié sous conditions :

- délai minimal entre deux changements ;
- nombre limité de changements ;
- vérification renforcée ;
- conservation temporaire de l’ancien identifiant pour éviter l’usurpation.

### 9.5 Confidentialité

L’utilisateur choisit qui peut le trouver :

- tout le monde ;
- contacts uniquement ;
- identifiant uniquement ;
- personne ;
- règles personnalisées.

## 10. Création du code de sécurité

### 10.1 PIN

Le PIN sert à :

- ouvrir l’application ;
- confirmer certaines actions ;
- sécuriser le compte.

Règles :

- longueur configurable ;
- interdiction des suites évidentes ;
- interdiction des répétitions trop simples ;
- confirmation du PIN ;
- stockage sécurisé par le système ;
- jamais transmis en clair.

### 10.2 Mot de passe

Selon l’architecture retenue, un mot de passe peut être utilisé pour :

- récupération ;
- connexion web ;
- changement d’appareil ;
- sécurité renforcée.

Il ne doit pas remplacer le PIN local.

### 10.3 Biométrie

Après création du PIN, proposer :

- Face ID ;
- Touch ID ;
- empreinte ;
- biométrie Android.

L’activation reste facultative.

## 11. Appareil de confiance

L’application enregistre :

- identifiant de l’appareil ;
- système ;
- version ;
- date d’ajout ;
- dernière utilisation ;
- risque ;
- statut.

L’utilisateur peut :

- voir les appareils ;
- renommer un appareil ;
- déconnecter un appareil ;
- bloquer tous les autres appareils.

Un nouvel appareil doit déclencher une notification.

## 12. KYC progressif

### 12.1 Principe

Le KYC ne doit pas être présenté comme un obstacle unique. Il doit fonctionner par niveaux.

### 12.2 Exemple de niveaux

#### Niveau 0 — Compte créé

Accès limité :

- consultation ;
- mode Découverte ;
- profil ;
- certaines fonctions non financières.

#### Niveau 1 — Téléphone vérifié

Accès possible :

- réception limitée ;
- Mansa Connect ;
- demandes d’argent ;
- limites faibles.

#### Niveau 2 — Identité déclarée

Informations :

- nom ;
- prénom ;
- date de naissance ;
- nationalité ;
- adresse.

Accès :

- limites supérieures ;
- paiements élargis.

#### Niveau 3 — Identité vérifiée

Documents :

- pièce d’identité ;
- selfie ;
- preuve de vie ;
- justificatif selon pays.

Accès :

- cartes ;
- virements ;
- limites supérieures ;
- services partenaires.

#### Niveau renforcé

Pour :

- gros montants ;
- investissements ;
- activités à risque ;
- exigences réglementaires.

### 12.3 Barre de progression

L’utilisateur voit :

- niveau actuel ;
- fonctions disponibles ;
- fonctions bloquées ;
- étapes restantes ;
- avantages de la vérification.

## 13. Informations personnelles

Champs possibles :

- prénom ;
- nom ;
- date de naissance ;
- genre uniquement si nécessaire ;
- nationalité ;
- pays de naissance ;
- résidence ;
- adresse ;
- profession ;
- source de revenus ;
- statut étudiant ;
- situation fiscale.

Chaque champ doit avoir :

- justification ;
- caractère obligatoire ou non ;
- règle par pays ;
- durée de conservation.

## 14. Vérification documentaire

### 14.1 Types de documents

Selon le pays :

- carte nationale ;
- passeport ;
- titre de séjour ;
- permis si autorisé ;
- document étudiant ;
- autre document accepté.

### 14.2 Capture

Fonctions :

- guide visuel ;
- détection des bords ;
- contrôle du flou ;
- contrôle de luminosité ;
- recto/verso ;
- vérification de cohérence ;
- reprise.

### 14.3 Selfie et preuve de vie

Le parcours peut demander :

- selfie ;
- mouvement de tête ;
- clignement ;
- courte vidéo ;
- comparaison faciale.

Une alternative manuelle doit exister si l’automatisation échoue.

### 14.4 Revue manuelle

Statuts :

- en attente ;
- en analyse ;
- accepté ;
- rejeté ;
- complément demandé ;
- vérification renforcée.

## 15. Import des contacts

### 15.1 Consentement

L’application explique :

> Vos contacts servent à retrouver les personnes qui utilisent Mansa. Ils ne seront pas publiés.

### 15.2 Refus

Le refus ne doit pas bloquer l’inscription.

### 15.3 Résultats

Afficher :

- contacts utilisant Mansa ;
- invitations possibles ;
- suggestions ;
- option de ne plus importer.

### 15.4 Confidentialité

Les contacts doivent être :

- hachés ou protégés lorsque possible ;
- comparés de manière sécurisée ;
- supprimables ;
- non utilisés pour le marketing sans consentement.

## 16. Personnalisation initiale

Étapes facultatives :

- photo ;
- nom affiché ;
- thème clair/sombre/système ;
- devise principale ;
- langue ;
- widgets ;
- raccourcis ;
- catégories favorites ;
- notifications ;
- visibilité du solde ;
- sons ;
- haptics.

L’utilisateur peut ignorer cette étape.

## 17. Première arrivée sur l’accueil

### 17.1 État initial

L’accueil peut afficher :

- solde réel ou vide ;
- niveau KYC ;
- actions rapides ;
- carte en attente ;
- identifiant Mansa ;
- contacts disponibles ;
- tutoriel ;
- mode Découverte ;
- Jini ;
- première mission.

### 17.2 Missions de démarrage

Exemples :

- sécuriser le compte ;
- choisir son identifiant ;
- ajouter une photo ;
- vérifier son identité ;
- inviter un contact ;
- effectuer une démonstration ;
- créer un coffre.

Les missions ne doivent pas infantiliser l’utilisateur.

## 18. Notifications d’onboarding

Notifications possibles :

- téléphone vérifié ;
- compte créé ;
- nouvel appareil ;
- KYC en attente ;
- KYC accepté ;
- complément demandé ;
- identifiant modifié ;
- biométrie activée ;
- premier contact trouvé ;
- onboarding incomplet.

Les notifications marketing restent séparées.

## 19. Administration

L’administration doit pouvoir configurer :

- pays ouverts ;
- langues ;
- canaux OTP ;
- champs obligatoires ;
- niveaux KYC ;
- documents acceptés ;
- seuils ;
- délais ;
- textes ;
- écrans de bienvenue ;
- disponibilité du mode Découverte ;
- fonctionnalités accessibles avant KYC ;
- règles d’âge ;
- règles de compte jeune ;
- maintenance ;
- versions minimales.

Toute modification doit être auditée.

## 20. Contrats API principaux

Exemples de routes à documenter précisément dans le contrat API final :

```http
POST /auth/register/start
POST /auth/register/verify-otp
POST /auth/register/complete
POST /auth/login
POST /auth/pin/setup
POST /auth/biometric/register
GET  /identity/username/check
POST /identity/username/reserve
PATCH /identity/username
POST /kyc/start
POST /kyc/documents
POST /kyc/liveness
GET  /kyc/status
POST /contacts/discovery
POST /devices/trust
GET  /devices
DELETE /devices/{deviceId}
POST /demo/session
DELETE /demo/session
```

Chaque route devra préciser :

- authentification ;
- permissions ;
- idempotence ;
- payload ;
- validation ;
- réponse ;
- erreurs ;
- audit ;
- rate limit.

## 21. Modèle de données attendu

Entités principales :

- User ;
- UserProfile ;
- UserIdentifier ;
- PhoneVerification ;
- EmailVerification ;
- RegistrationSession ;
- Consent ;
- Device ;
- TrustedDevice ;
- SecurityCredential ;
- KycProfile ;
- KycDocument ;
- KycReview ;
- ContactDiscoveryConsent ;
- OnboardingProgress ;
- DemoSession ;
- UserPreference ;
- NotificationPreference.

Les schémas Prisma complets seront documentés dans un fichier technique dédié.

## 22. Règles métier principales

1. Un numéro ne peut être associé qu’à un seul compte actif, sauf cas spécial validé.
2. Un identifiant Mansa doit être unique.
3. Une session d’inscription expire après un délai configurable.
4. Un OTP expiré ne peut jamais être réutilisé.
5. Les fonctionnalités disponibles dépendent du niveau KYC.
6. Le mode Découverte ne peut jamais créer de transaction réelle.
7. L’utilisateur peut refuser l’import des contacts.
8. Un compte mineur doit respecter les règles du compte jeune.
9. Un nouvel appareil déclenche une vérification renforcée selon le risque.
10. Toute modification sensible doit être auditée.
11. Le consentement marketing ne doit jamais être obligatoire.
12. Les documents rejetés doivent avoir une raison compréhensible.
13. Une reprise d’onboarding ne doit pas recréer un second compte.
14. Les erreurs techniques ne doivent pas révéler d’informations sensibles.
15. Les limites et exigences varient selon le pays et le partenaire.

## 23. Analytics

Événements possibles :

- onboarding_started ;
- country_selected ;
- language_selected ;
- registration_phone_submitted ;
- otp_requested ;
- otp_verified ;
- username_selected ;
- pin_created ;
- biometric_enabled ;
- kyc_started ;
- kyc_completed ;
- kyc_failed ;
- contacts_permission_granted ;
- contacts_permission_denied ;
- demo_started ;
- demo_completed ;
- onboarding_completed ;
- onboarding_abandoned.

Aucun événement ne doit contenir :

- OTP ;
- PIN ;
- document brut ;
- photo brute ;
- secret ;
- donnée bancaire sensible.

## 24. Cas d’erreur

Le document final doit couvrir :

- aucune connexion ;
- connexion instable ;
- backend indisponible ;
- OTP non reçu ;
- téléphone déjà utilisé ;
- e-mail déjà utilisé ;
- identifiant indisponible ;
- appareil compromis ;
- version obsolète ;
- pays non supporté ;
- utilisateur mineur ;
- document illisible ;
- selfie rejeté ;
- doublon d’identité ;
- compte suspendu ;
- KYC manuel ;
- abandon puis reprise ;
- changement de téléphone ;
- horloge de l’appareil incorrecte.

## 25. Critères d’acceptation

Le module est considéré comme prêt lorsque :

- l’utilisateur peut créer un compte sans ambiguïté ;
- le parcours reprend après interruption ;
- les consentements sont séparés ;
- l’OTP est sécurisé ;
- l’identifiant Mansa est unique ;
- le PIN et la biométrie fonctionnent ;
- les niveaux KYC sont appliqués ;
- le refus des contacts ne bloque pas ;
- le mode Découverte est isolé ;
- les appareils sont visibles et révocables ;
- les erreurs sont compréhensibles ;
- l’accessibilité est testée ;
- les événements analytics sont conformes ;
- l’administration peut modifier les règles prévues ;
- les tests automatisés principaux passent.

## 26. Tests attendus

### Tests unitaires

- validation téléphone ;
- validation e-mail ;
- expiration OTP ;
- unicité identifiant ;
- niveaux KYC ;
- règles d’âge ;
- progression onboarding ;
- isolation du mode Découverte.

### Tests d’intégration

- inscription complète ;
- reprise ;
- changement de canal OTP ;
- création appareil ;
- démarrage KYC ;
- complément documentaire ;
- suppression appareil.

### Tests end-to-end

- utilisateur standard ;
- utilisateur mineur ;
- pays non disponible ;
- connexion faible ;
- OTP expiré ;
- identifiant déjà pris ;
- KYC accepté ;
- KYC rejeté ;
- mode Découverte ;
- activation biométrie.
