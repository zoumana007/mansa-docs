# Authentification, connexion, récupération de compte et sessions

## 1. Objectif

Ce module doit permettre à l’utilisateur de :

- se connecter rapidement ;
- protéger son compte ;
- récupérer son accès sans assistance inutile ;
- contrôler tous ses appareils ;
- détecter les connexions suspectes ;
- bloquer immédiatement une session compromise ;
- continuer à utiliser l’application même après un changement de téléphone.

L’expérience doit rester simple pour l’utilisateur légitime et difficile à contourner pour un fraudeur.

## 2. Méthodes de connexion

Les méthodes disponibles dépendent du pays, du niveau de sécurité, de l’appareil et du risque.

Méthodes possibles :

- numéro de téléphone + PIN ;
- e-mail + mot de passe ;
- biométrie locale ;
- lien sécurisé à usage unique ;
- code OTP ;
- connexion via appareil déjà approuvé ;
- clé d’accès ou passkey lorsque compatible.

La biométrie ne remplace pas l’identité du compte. Elle sert à déverrouiller localement une session déjà autorisée.

## 3. Écran de connexion

L’écran doit contenir :

- logo Mansa ;
- pays ou indicatif ;
- numéro ou e-mail ;
- bouton Continuer ;
- lien « J’ai oublié mon accès » ;
- lien « Créer un compte » ;
- accès au support ;
- changement de langue ;
- indication du mode Démonstration lorsque disponible.

L’application ne doit pas confirmer publiquement qu’un numéro ou e-mail existe avant la vérification de sécurité appropriée.

## 4. Connexion par téléphone

Parcours :

1. saisie du numéro ;
2. validation du format ;
3. détection d’un appareil connu ;
4. demande du PIN ou d’une vérification supplémentaire ;
5. ouverture de session ;
6. notification de connexion.

Selon le risque, l’application peut demander :

- OTP ;
- biométrie ;
- confirmation depuis un appareil existant ;
- selfie ou preuve de vie ;
- vérification KYC renforcée ;
- délai de sécurité.

## 5. Connexion par biométrie

La biométrie peut être utilisée uniquement si :

- l’appareil a déjà été approuvé ;
- une session sécurisée existe ;
- la biométrie du système est disponible ;
- l’utilisateur l’a activée ;
- aucun signal de risque élevé n’est détecté.

Cas d’échec :

- trop de tentatives ;
- changement des empreintes ou du visage enregistrés ;
- biométrie désactivée ;
- appareil compromis ;
- session expirée.

Dans ces cas, retour au PIN ou à une vérification renforcée.

## 6. PIN

Le PIN peut servir à :

- ouvrir l’application ;
- confirmer certaines opérations ;
- modifier certains paramètres ;
- autoriser un paiement selon les règles.

Règles :

- longueur configurable ;
- nombre de tentatives limité ;
- délai progressif après erreur ;
- blocage temporaire ;
- réinitialisation sécurisée ;
- stockage uniquement dans le coffre sécurisé de l’appareil ;
- jamais enregistré en clair côté serveur.

Le PIN de connexion locale et le code PIN d’une carte bancaire doivent être clairement distingués.

## 7. Mot de passe

Le mot de passe peut être utilisé pour :

- connexion web ;
- récupération ;
- changement d’appareil ;
- opérations de sécurité particulières.

Exigences :

- longueur minimale ;
- contrôle contre les mots de passe compromis ;
- interdiction des mots de passe trop courants ;
- historique éventuel ;
- limitation des essais ;
- changement volontaire ;
- invalidation des sessions si nécessaire.

L’application doit privilégier la robustesse sans imposer des règles inutiles comme des combinaisons artificielles difficiles à retenir.

## 8. Passkeys

Lorsque l’appareil et le pays le permettent, Mansa peut proposer une passkey.

Avantages :

- pas de mot de passe à mémoriser ;
- résistance au phishing ;
- authentification liée à l’appareil ou au compte système ;
- biométrie intégrée.

L’utilisateur doit toujours disposer d’une méthode de récupération alternative.

## 9. Appareil connu et appareil inconnu

### Appareil connu

Connexion simplifiée possible avec :

- biométrie ;
- PIN ;
- vérification légère.

### Nouvel appareil

Vérifications possibles :

- OTP ;
- e-mail ;
- confirmation depuis l’ancien appareil ;
- code de récupération ;
- preuve de vie ;
- délai de sécurité ;
- questions contextuelles limitées ;
- contrôle du risque.

Le nouvel appareil ne doit pas recevoir immédiatement tous les privilèges si le risque est élevé.

## 10. Approbation depuis un appareil existant

Lorsqu’un utilisateur se connecte sur un nouveau téléphone, l’ancien appareil peut recevoir :

> Une tentative de connexion a été détectée.

Actions :

- Autoriser ;
- Refuser ;
- Ce n’est pas moi ;
- Voir les détails.

Détails affichés :

- type d’appareil ;
- ville approximative ;
- pays ;
- heure ;
- adresse IP partiellement masquée ;
- version de l’application.

Une approbation ne doit jamais être automatique.

## 11. Gestion des sessions

L’utilisateur doit pouvoir consulter :

- appareils actifs ;
- sessions ouvertes ;
- date de dernière activité ;
- localisation approximative ;
- navigateur ou système ;
- niveau de confiance ;
- statut.

Actions possibles :

- renommer l’appareil ;
- fermer une session ;
- fermer toutes les autres sessions ;
- bloquer l’appareil ;
- signaler une connexion suspecte.

## 12. Durée des sessions

La durée dépend :

- du type d’appareil ;
- de la sensibilité du compte ;
- du niveau KYC ;
- du risque ;
- du type d’action.

Exemples :

- session applicative longue avec renouvellement sécurisé ;
- réauthentification pour une opération sensible ;
- expiration rapide après inactivité sur un appareil non fiable ;
- fermeture immédiate après changement critique.

## 13. Réauthentification

Certaines actions doivent demander une confirmation supplémentaire :

- modification du téléphone ;
- modification de l’e-mail ;
- changement de mot de passe ;
- changement de PIN ;
- ajout d’un nouvel appareil ;
- changement d’identifiant Mansa ;
- augmentation de plafond ;
- ajout d’un bénéficiaire ;
- gros transfert ;
- désactivation de la biométrie ;
- export de données ;
- fermeture du compte.

Méthodes possibles :

- biométrie ;
- PIN ;
- OTP ;
- passkey ;
- confirmation sur autre appareil ;
- vérification renforcée.

## 14. Mot de passe oublié

Parcours :

1. saisie du téléphone ou e-mail ;
2. vérification anti-abus ;
3. OTP ou lien sécurisé ;
4. contrôle du risque ;
5. création d’un nouveau mot de passe ;
6. invalidation des anciennes sessions selon le niveau de risque ;
7. notification de sécurité.

Le système ne doit pas permettre de contourner les protections simplement avec l’accès au téléphone.

## 15. PIN oublié

Parcours possible :

1. démarrer la récupération ;
2. vérifier le téléphone ;
3. confirmer l’identité ;
4. effectuer une vérification renforcée ;
5. définir un nouveau PIN ;
6. révoquer les jetons locaux précédents ;
7. notifier l’utilisateur.

Selon le risque, un délai de sécurité peut être appliqué avant le retour complet des fonctions sensibles.

## 16. Téléphone perdu ou volé

L’utilisateur doit pouvoir :

- se connecter depuis un autre appareil ;
- utiliser un portail de récupération ;
- contacter le support ;
- fermer toutes les sessions ;
- bloquer les cartes ;
- désactiver temporairement les paiements ;
- verrouiller le compte ;
- révoquer l’ancien appareil.

Un bouton d’urgence doit permettre de sécuriser rapidement le compte.

## 17. Changement de numéro

Le changement doit exiger :

- authentification récente ;
- vérification de l’ancien numéro lorsque possible ;
- vérification du nouveau numéro ;
- contrôle de risque ;
- délai de sécurité selon le profil ;
- notification sur tous les canaux ;
- journal d’audit.

Si l’ancien numéro est inaccessible, un parcours renforcé doit être utilisé.

## 18. Changement d’e-mail

Même principe :

- réauthentification ;
- confirmation de l’ancien e-mail si possible ;
- confirmation du nouveau ;
- notification ;
- journalisation ;
- délai de sécurité éventuel.

## 19. Compte verrouillé

Le compte peut être verrouillé en cas de :

- trop nombreuses tentatives ;
- fraude suspectée ;
- appareil compromis ;
- demande utilisateur ;
- obligation réglementaire ;
- incident de sécurité.

Statuts possibles :

- verrouillage temporaire ;
- accès limité ;
- paiements suspendus ;
- compte suspendu ;
- fermeture en cours.

L’utilisateur doit recevoir une explication claire, sauf si une règle légale interdit de la communiquer.

## 20. Détection de risque

Signaux possibles :

- nouvel appareil ;
- nouveau pays ;
- adresse IP inhabituelle ;
- émulateur ou appareil rooté ;
- version modifiée de l’application ;
- tentatives répétées ;
- vitesse anormale de navigation ;
- changement brutal de comportement ;
- carte SIM récemment changée ;
- compte récemment récupéré ;
- accès depuis plusieurs pays rapprochés ;
- données de sécurité incohérentes.

Le score de risque peut déclencher :

- autorisation ;
- vérification supplémentaire ;
- limitation temporaire ;
- blocage ;
- revue manuelle.

## 21. Protection contre le phishing

Mansa doit :

- ne jamais demander le PIN complet par message ;
- ne jamais demander un OTP dans une conversation ;
- afficher clairement les messages officiels ;
- utiliser des domaines et liens contrôlés ;
- avertir avant l’ouverture d’un lien externe ;
- proposer un centre de vérification des communications ;
- permettre de signaler un message suspect.

Une notification officielle doit être distinguable d’un message envoyé par un utilisateur.

## 22. Notifications de sécurité

Envoyer une alerte lors de :

- nouvelle connexion ;
- nouvel appareil ;
- mot de passe modifié ;
- PIN réinitialisé ;
- biométrie activée ou désactivée ;
- numéro modifié ;
- e-mail modifié ;
- session révoquée ;
- récupération lancée ;
- trop nombreuses tentatives ;
- connexion suspecte ;
- fermeture globale des sessions.

Canaux :

- push ;
- e-mail ;
- SMS pour les événements critiques ;
- notification dans l’application.

## 23. Écran Centre de sécurité

Le Centre de sécurité doit regrouper :

- appareils ;
- sessions ;
- PIN ;
- mot de passe ;
- passkeys ;
- biométrie ;
- double authentification ;
- alertes ;
- historique de sécurité ;
- contacts de récupération ;
- bouton d’urgence ;
- conseils personnalisés.

Il peut afficher un niveau de sécurité :

- Faible ;
- Moyen ;
- Bon ;
- Renforcé.

## 24. Contacts de récupération

Option facultative permettant de désigner un contact de confiance.

Ce contact ne peut pas accéder au compte ou à l’argent.

Il peut uniquement participer à une procédure de récupération encadrée.

Protections :

- délai d’activation ;
- confirmation des deux parties ;
- retrait possible ;
- journalisation ;
- impossibilité d’autoriser seul un transfert.

## 25. Codes de récupération

Mansa peut générer des codes à usage unique.

Règles :

- affichés une seule fois ;
- stockés hachés ;
- révocables ;
- utilisables une seule fois ;
- renouvelables ;
- protégés par réauthentification.

## 26. Administration

L’administration doit pouvoir configurer :

- méthodes de connexion ;
- durée des sessions ;
- seuils de risque ;
- règles de réauthentification ;
- nombre d’essais ;
- délais de blocage ;
- canaux de récupération ;
- règles par pays ;
- règles selon le niveau KYC ;
- versions minimales ;
- appareils interdits ;
- politiques de passkey ;
- délais de sécurité.

Les administrateurs ne doivent jamais pouvoir voir un mot de passe, un PIN, une passkey ou un OTP.

## 27. Contrats API principaux

```http
POST   /auth/login/start
POST   /auth/login/verify
POST   /auth/login/complete
POST   /auth/logout
POST   /auth/logout-all
POST   /auth/token/refresh
POST   /auth/password/reset/start
POST   /auth/password/reset/verify
POST   /auth/password/reset/complete
POST   /auth/pin/reset/start
POST   /auth/pin/reset/complete
POST   /auth/passkeys/register
POST   /auth/passkeys/authenticate
GET    /security/sessions
DELETE /security/sessions/{sessionId}
DELETE /security/sessions
GET    /security/devices
PATCH  /security/devices/{deviceId}
DELETE /security/devices/{deviceId}
POST   /security/devices/{deviceId}/approve
POST   /security/emergency-lock
GET    /security/events
```

Chaque contrat doit préciser :

- authentification ;
- idempotence ;
- limitation de débit ;
- données requises ;
- réponse ;
- erreurs ;
- audit ;
- niveau de risque ;
- réauthentification éventuelle.

## 28. Modèles de données attendus

- AuthSession ;
- RefreshToken ;
- LoginAttempt ;
- TrustedDevice ;
- DeviceApproval ;
- PasswordCredential ;
- PinCredential ;
- PasskeyCredential ;
- RecoveryCode ;
- RecoveryRequest ;
- RecoveryContact ;
- SecurityEvent ;
- RiskAssessment ;
- AccountLock ;
- AuthenticationChallenge ;
- OtpChallenge.

## 29. Règles métier principales

1. Une session doit pouvoir être révoquée immédiatement.
2. Un jeton de rafraîchissement réutilisé doit être considéré comme suspect.
3. Un OTP expiré ne peut pas être réutilisé.
4. Une passkey supprimée doit devenir inutilisable immédiatement.
5. Une connexion sur nouvel appareil peut nécessiter une approbation.
6. Une récupération de compte doit augmenter temporairement le niveau de surveillance.
7. Un changement de téléphone peut déclencher un délai de sécurité.
8. Un administrateur ne peut jamais obtenir les secrets d’authentification.
9. Les événements de sécurité critiques doivent être journalisés.
10. La biométrie locale ne doit jamais être envoyée au serveur.
11. Une session fermée ne doit plus permettre de renouveler un jeton.
12. Un utilisateur doit pouvoir fermer toutes ses sessions.
13. Les messages d’erreur ne doivent pas révéler l’existence d’un compte.
14. Les opérations sensibles exigent une authentification récente.
15. Le support ne peut pas contourner seul une récupération renforcée.

## 30. Cas d’erreur

- mauvais PIN ;
- mot de passe incorrect ;
- OTP expiré ;
- compte inexistant ;
- compte suspendu ;
- appareil bloqué ;
- session expirée ;
- jeton révoqué ;
- passkey inconnue ;
- biométrie indisponible ;
- trop de tentatives ;
- changement de SIM ;
- téléphone perdu ;
- e-mail inaccessible ;
- récupération en cours ;
- approbation refusée ;
- serveur indisponible ;
- connexion faible ;
- horloge incorrecte ;
- appareil rooté ;
- session concurrente suspecte.

## 31. Critères d’acceptation

Le module est prêt lorsque :

- un utilisateur peut se connecter sur appareil connu ;
- un nouvel appareil est soumis aux vérifications prévues ;
- la biométrie fonctionne sans exposer de données biométriques ;
- la récupération de mot de passe fonctionne ;
- la récupération de PIN fonctionne ;
- toutes les sessions sont visibles ;
- chaque session peut être révoquée ;
- le bouton d’urgence sécurise le compte ;
- les événements critiques déclenchent des notifications ;
- les erreurs restent neutres et sûres ;
- les règles de risque sont configurables ;
- les jetons révoqués deviennent inutilisables ;
- les tests d’authentification et de récupération passent.

## 32. Tests attendus

### Tests unitaires

- validation des identifiants ;
- rotation des jetons ;
- expiration des sessions ;
- compteur de tentatives ;
- blocage temporaire ;
- validation des passkeys ;
- utilisation unique des codes de récupération ;
- score de risque ;
- vérification de l’authentification récente.

### Tests d’intégration

- connexion complète ;
- renouvellement de session ;
- fermeture d’une session ;
- fermeture globale ;
- nouvel appareil ;
- approbation distante ;
- récupération de mot de passe ;
- récupération de PIN ;
- changement de téléphone ;
- verrouillage d’urgence.

### Tests end-to-end

- connexion normale ;
- biométrie refusée ;
- OTP expiré ;
- appareil inconnu ;
- appareil compromis ;
- compte suspendu ;
- mot de passe oublié ;
- téléphone perdu ;
- passkey ;
- fermeture de toutes les sessions ;
- tentative de réutilisation d’un jeton révoqué.
