# Profil utilisateur, identité Mansa, confidentialité et préférences

## 1. Objectif

Ce module doit permettre à l’utilisateur de :

- gérer son identité dans Mansa ;
- contrôler les informations visibles par les autres ;
- modifier ses préférences ;
- utiliser son identifiant unique Mansa ;
- partager son profil sans révéler son numéro de téléphone ;
- contrôler sa présence dans Mansa Connect ;
- personnaliser l’application ;
- exercer ses droits sur ses données.

Le profil Mansa ne doit pas être un simple formulaire. Il doit devenir l’identité numérique de l’utilisateur dans tout l’écosystème.

## 2. Structure du profil

Le profil peut contenir :

- photo ;
- prénom ;
- nom ;
- nom affiché ;
- identifiant unique Mansa ;
- numéro de téléphone masqué ;
- e-mail masqué ;
- pays ;
- ville facultative ;
- langue ;
- photo de couverture facultative ;
- courte biographie facultative ;
- QR personnel ;
- lien de profil ;
- badges de vérification ;
- statut du compte ;
- niveau KYC ;
- préférences de contact ;
- paramètres de confidentialité.

Les informations légales vérifiées et les informations publiques doivent être clairement séparées.

## 3. Identité légale et identité publique

### 3.1 Identité légale

Elle contient les informations utilisées pour :

- KYC ;
- conformité ;
- contrats ;
- paiements ;
- fiscalité ;
- partenaires bancaires.

Exemples :

- nom légal ;
- prénom légal ;
- date de naissance ;
- nationalité ;
- adresse ;
- document d’identité.

Ces informations ne doivent pas être modifiables librement sans vérification.

### 3.2 Identité publique

Elle contient ce que les autres peuvent voir selon les réglages :

- nom affiché ;
- photo ;
- identifiant Mansa ;
- badge ;
- courte biographie ;
- pays ou ville facultative ;
- statut de disponibilité ;
- QR de profil.

L’utilisateur peut personnaliser cette partie sans modifier son identité légale.

## 4. Identifiant unique Mansa

Chaque utilisateur possède un identifiant unique.

Exemples :

- `@zoumana` ;
- `@camara007` ;
- `@mansa-zou`.

Cet identifiant sert à :

- rechercher un utilisateur ;
- ouvrir une conversation ;
- envoyer de l’argent ;
- demander de l’argent ;
- ajouter un bénéficiaire ;
- partager un profil ;
- rejoindre une cagnotte ;
- recevoir un paiement ;
- partager un lien Mansa ;
- utiliser Mansa Connect sans communiquer son numéro.

## 5. Règles de l’identifiant

L’identifiant doit :

- être unique ;
- être insensible à la casse ;
- être normalisé ;
- respecter une longueur minimale et maximale ;
- utiliser uniquement les caractères autorisés ;
- ne pas imiter une institution ;
- ne pas contenir de terme offensant ;
- ne pas être trompeur ;
- ne pas être réservé ;
- ne pas utiliser un identifiant déjà protégé.

L’administration doit pouvoir gérer :

- mots interdits ;
- identifiants réservés ;
- identifiants institutionnels ;
- contestations ;
- usurpations ;
- blocages.

## 6. Changement d’identifiant

Le changement doit respecter :

- délai minimal entre deux modifications ;
- nombre limité de changements ;
- authentification récente ;
- notification de sécurité ;
- journal d’audit ;
- période de protection de l’ancien identifiant ;
- interdiction temporaire de réutilisation par un autre utilisateur.

Si un ancien identifiant est partagé dans une conversation ou un reçu, il doit continuer à renvoyer vers le bon historique sans créer de confusion.

## 7. Carte de Profil Mansa

Chaque utilisateur possède une carte de profil numérique.

Elle peut contenir :

- photo ;
- nom affiché ;
- identifiant Mansa ;
- QR ;
- badge ;
- pays ;
- langue ;
- bouton Envoyer ;
- bouton Demander ;
- bouton Message ;
- bouton Ajouter aux favoris ;
- bouton Partager ;
- bouton Bloquer ;
- bouton Signaler.

Cette carte peut être partagée par :

- QR ;
- NFC ;
- lien ;
- messagerie ;
- AirDrop ou partage système ;
- copie de l’identifiant.

## 8. QR personnel

Le profil doit proposer plusieurs QR.

### QR permanent

Il ouvre le profil ou la conversation.

### QR de paiement

Il permet de recevoir un montant.

### QR avec montant prérempli

Exemple :

- Recevoir 25 000 FCFA.

### QR temporaire

Il expire après une durée.

### QR à usage unique

Il devient invalide après utilisation.

### QR privé

Il fonctionne uniquement pour certaines personnes ou dans un groupe donné.

## 9. Partage par NFC

Deux utilisateurs peuvent rapprocher leurs téléphones pour :

- partager leur profil ;
- ouvrir une conversation ;
- ajouter un bénéficiaire ;
- partager une demande de paiement ;
- rejoindre une cagnotte.

L’échange NFC ne doit jamais déclencher automatiquement un transfert d’argent.

Une confirmation explicite reste obligatoire.

## 10. Recherche d’un utilisateur

Un utilisateur peut être retrouvé par :

- identifiant Mansa ;
- nom affiché ;
- numéro enregistré dans les contacts ;
- QR ;
- NFC ;
- lien de profil ;
- bénéficiaire récent ;
- suggestion Mansa Connect.

La recherche doit respecter les paramètres de confidentialité.

## 11. Paramètres de visibilité

L’utilisateur peut choisir qui peut le trouver :

- tout le monde ;
- contacts uniquement ;
- personnes ayant son identifiant ;
- personnes ayant son QR ;
- aucun utilisateur ;
- règles personnalisées.

Il peut aussi définir séparément qui peut :

- envoyer un message ;
- demander de l’argent ;
- envoyer de l’argent ;
- l’ajouter à un groupe ;
- l’inviter à une cagnotte ;
- voir sa photo ;
- voir son nom complet ;
- voir sa ville ;
- voir son statut en ligne ;
- voir la confirmation de lecture.

## 12. Nom affiché

Le nom affiché peut être :

- prénom ;
- prénom + initiale ;
- nom complet ;
- pseudonyme ;
- nom personnalisé autorisé.

Pour les opérations financières, l’application doit afficher suffisamment d’informations vérifiées pour éviter les erreurs de destinataire.

Exemple :

- Zoumana C. ;
- Compte vérifié ;
- `@mansa-zou`.

## 13. Badges

Badges possibles :

- particulier vérifié ;
- commerçant vérifié ;
- banque partenaire ;
- institution publique ;
- université ;
- association ;
- entreprise ;
- créateur vérifié ;
- partenaire officiel.

Les badges ne doivent jamais être attribués manuellement sans procédure définie.

Chaque badge doit avoir :

- type ;
- date d’attribution ;
- organisme ;
- statut ;
- date d’expiration éventuelle ;
- preuve de validation.

## 14. Statut et présence

L’utilisateur peut choisir :

- En ligne ;
- Disponible ;
- Occupé ;
- Ne pas déranger ;
- Invisible ;
- Statut personnalisé.

Le statut ne doit jamais révéler précisément la localisation ou l’activité financière.

## 15. Photo de profil

Fonctions :

- prendre une photo ;
- importer une image ;
- recadrer ;
- supprimer ;
- remplacer ;
- masquer pour certains utilisateurs.

Contrôles :

- format ;
- taille ;
- contenu interdit ;
- nudité ou violence ;
- usurpation ;
- modération ;
- compression.

## 16. Biographie

La biographie est facultative.

Règles :

- longueur limitée ;
- pas de données bancaires sensibles ;
- pas de contenu interdit ;
- liens limités ou contrôlés ;
- modération ;
- signalement possible.

## 17. Favoris et contacts Mansa

L’utilisateur peut :

- ajouter un profil aux favoris ;
- créer des groupes de favoris ;
- renommer localement un contact ;
- ajouter une note privée ;
- épingler un bénéficiaire ;
- masquer une suggestion ;
- retirer un favori.

Les notes privées ne sont jamais visibles par l’autre utilisateur.

## 18. Blocage

Bloquer un utilisateur doit empêcher :

- nouveaux messages ;
- nouvelles demandes d’argent ;
- invitations à des groupes ;
- affichage du statut ;
- recherche directe selon les règles ;
- interactions non autorisées.

Le blocage ne doit pas supprimer :

- historique financier ;
- preuves de paiement ;
- reçus ;
- litiges ;
- obligations légales.

## 19. Signalement

Motifs possibles :

- fraude ;
- usurpation ;
- harcèlement ;
- spam ;
- contenu interdit ;
- tentative de phishing ;
- demande d’argent abusive ;
- faux profil ;
- autre.

Le signalement doit produire :

- identifiant du dossier ;
- date ;
- catégorie ;
- contenu concerné ;
- statut ;
- suivi.

## 20. Préférences générales

L’utilisateur peut configurer :

- langue ;
- pays d’affichage ;
- devise principale ;
- thème ;
- apparence ;
- taille du texte ;
- animations ;
- sons ;
- vibrations ;
- haptics ;
- écran d’accueil ;
- widgets ;
- raccourcis ;
- visibilité du solde ;
- format des dates ;
- fuseau horaire ;
- unités ;
- mode faible consommation ;
- mode connexion lente.

## 21. Préférences de notifications

Catégories :

- paiements ;
- transferts ;
- cartes ;
- sécurité ;
- messagerie ;
- demandes d’argent ;
- cagnottes ;
- promotions ;
- fidélité ;
- services publics ;
- Jini ;
- investissements ;
- documents ;
- maintenance ;
- actualités.

Pour chaque catégorie, l’utilisateur peut choisir :

- push ;
- e-mail ;
- SMS ;
- notification dans l’application ;
- aucune notification, sauf obligations critiques.

Les alertes de sécurité critiques ne doivent pas pouvoir être désactivées entièrement.

## 22. Confidentialité de Mansa Connect

Réglages possibles :

- accusés de lecture ;
- statut en ligne ;
- indicateur de saisie ;
- photo visible ;
- ajout aux groupes ;
- réception de demandes ;
- appels futurs ;
- messages de personnes inconnues ;
- filtres anti-spam.

## 23. Personnalisation de l’accueil

L’utilisateur peut choisir :

- ordre des widgets ;
- actions rapides ;
- cartes visibles ;
- solde masqué ou visible ;
- raccourcis Mansa Connect ;
- raccourcis paiements ;
- objectifs affichés ;
- recommandations Jini ;
- services favoris ;
- promotions visibles ;
- actualité financière.

L’administration peut imposer certains éléments critiques comme :

- alerte de sécurité ;
- maintenance ;
- obligation réglementaire ;
- demande KYC ;
- blocage de compte.

## 24. Profil jeune

Pour un compte jeune, certaines préférences peuvent être :

- supervisées ;
- verrouillées ;
- soumises à validation parentale ;
- limitées selon l’âge.

Le jeune doit néanmoins disposer d’un espace de confidentialité adapté.

Le parent ne doit pas accéder automatiquement à toutes les conversations privées sauf cadre légal ou politique produit clairement défini.

## 25. Profil étudiant

Le profil étudiant peut contenir :

- établissement ;
- statut étudiant ;
- carte étudiante ;
- identifiant universitaire ;
- année académique ;
- avantages ;
- bourse ;
- services disponibles.

Ces informations ne doivent être visibles publiquement que si l’utilisateur l’autorise.

## 26. Export des données

L’utilisateur doit pouvoir demander :

- export du profil ;
- historique des préférences ;
- consentements ;
- appareils ;
- sessions ;
- transactions selon les règles ;
- conversations selon la politique applicable ;
- documents disponibles ;
- données analytiques accessibles.

L’export doit être :

- sécurisé ;
- authentifié ;
- temporaire ;
- chiffré si nécessaire ;
- audité.

## 27. Suppression du compte

La suppression doit distinguer :

- désactivation ;
- fermeture ;
- suppression de données facultatives ;
- conservation légale obligatoire ;
- anonymisation ;
- suppression définitive lorsque permise.

Avant fermeture :

- afficher les soldes ;
- vérifier les opérations en attente ;
- fermer ou transférer les fonds ;
- traiter les litiges ;
- révoquer les cartes ;
- informer sur les données conservées.

## 28. Administration

L’administration doit pouvoir gérer :

- identifiants réservés ;
- badges ;
- profils signalés ;
- photos ;
- biographies ;
- règles de visibilité ;
- limites de modification ;
- règles de recherche ;
- politiques de blocage ;
- catégories de signalement ;
- langues ;
- préférences disponibles ;
- éléments imposés ;
- profils jeunes ;
- profils étudiants.

Les administrateurs ne doivent pas modifier silencieusement l’identité publique sans motif, permission et audit.

## 29. Contrats API principaux

```http
GET    /profile/me
PATCH  /profile/me
POST   /profile/avatar
DELETE /profile/avatar
GET    /profile/{identifier}
GET    /profile/search
GET    /identity/username/check
PATCH  /identity/username
GET    /profile/qr
POST   /profile/qr/temporary
POST   /profile/qr/one-time
GET    /preferences
PATCH  /preferences
GET    /privacy
PATCH  /privacy
POST   /contacts/favorites/{userId}
DELETE /contacts/favorites/{userId}
POST   /users/{userId}/block
DELETE /users/{userId}/block
POST   /users/{userId}/report
GET    /security/blocked-users
POST   /account/export
POST   /account/close
```

Chaque contrat doit préciser :

- authentification ;
- permissions ;
- visibilité ;
- limites ;
- erreurs ;
- audit ;
- rate limit ;
- consentement ;
- réauthentification.

## 30. Modèles de données attendus

- UserProfile ;
- PublicProfile ;
- UserIdentifier ;
- IdentifierHistory ;
- ProfileBadge ;
- ProfileVisibility ;
- UserPreference ;
- NotificationPreference ;
- PrivacyPreference ;
- FavoriteContact ;
- LocalContactAlias ;
- BlockedUser ;
- UserReport ;
- PersonalQrCode ;
- TemporaryProfileLink ;
- ProfileViewEvent ;
- AccountClosureRequest ;
- DataExportRequest.

## 31. Règles métier principales

1. L’identité légale et l’identité publique doivent être séparées.
2. L’identifiant Mansa doit rester unique.
3. Un ancien identifiant doit être protégé pendant une durée configurable.
4. Le numéro de téléphone ne doit pas être exposé sans consentement.
5. Le blocage ne supprime jamais les preuves financières.
6. Un badge doit être attribué selon une procédure vérifiable.
7. Les paramètres de confidentialité doivent être appliqués à la recherche.
8. Les alertes de sécurité critiques ne peuvent pas être totalement désactivées.
9. Une modification sensible doit être auditée.
10. Un profil supprimé doit respecter les obligations de conservation.
11. Un administrateur ne peut pas modifier un identifiant sans motif et permission.
12. Les notes privées sur un contact restent privées.
13. Un QR à usage unique devient invalide après utilisation.
14. Un QR temporaire expire automatiquement.
15. Le partage NFC ne déclenche aucune transaction sans confirmation.

## 32. Analytics

Événements possibles :

- profile_viewed ;
- profile_updated ;
- avatar_updated ;
- username_changed ;
- privacy_updated ;
- preference_updated ;
- profile_shared ;
- profile_qr_generated ;
- user_blocked ;
- user_unblocked ;
- user_reported ;
- favorite_added ;
- favorite_removed ;
- data_export_requested ;
- account_closure_started.

Les analytics ne doivent pas contenir :

- contenu privé ;
- note privée ;
- numéro complet ;
- e-mail complet ;
- document KYC ;
- information biométrique ;
- secret.

## 33. Cas d’erreur

- identifiant déjà utilisé ;
- identifiant réservé ;
- photo refusée ;
- profil introuvable ;
- profil privé ;
- utilisateur bloqué ;
- QR expiré ;
- QR déjà utilisé ;
- modification trop fréquente ;
- badge expiré ;
- export indisponible ;
- fermeture impossible à cause d’un solde ;
- compte sous litige ;
- profil suspendu ;
- connexion faible ;
- données non synchronisées.

## 34. Critères d’acceptation

Le module est prêt lorsque :

- l’utilisateur peut gérer son profil ;
- l’identité légale reste protégée ;
- l’identité publique est configurable ;
- l’identifiant Mansa fonctionne ;
- la recherche respecte la confidentialité ;
- les QR permanents, temporaires et uniques fonctionnent ;
- le blocage fonctionne ;
- le signalement crée un dossier ;
- les préférences sont synchronisées ;
- les notifications sont configurables ;
- l’export des données est sécurisé ;
- la fermeture du compte respecte les règles ;
- les modifications sensibles sont auditées ;
- les tests passent.

## 35. Tests attendus

### Tests unitaires

- unicité identifiant ;
- validation des caractères ;
- ancien identifiant protégé ;
- visibilité du profil ;
- expiration QR ;
- blocage ;
- filtrage de recherche ;
- préférences ;
- notifications critiques.

### Tests d’intégration

- modification du profil ;
- changement d’identifiant ;
- génération QR ;
- recherche ;
- ajout favori ;
- blocage ;
- signalement ;
- export ;
- fermeture du compte.

### Tests end-to-end

- profil public ;
- profil privé ;
- recherche par identifiant ;
- recherche par contact ;
- QR temporaire ;
- QR à usage unique ;
- utilisateur bloqué ;
- profil signalé ;
- changement de thème ;
- export des données ;
- fermeture impossible avec solde restant.
