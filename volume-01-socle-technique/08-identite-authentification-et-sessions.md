# Identité, authentification et sessions

## Objectif

Ce document définit le socle commun d’identité de Mansa. Il s’applique aux applications Client, Commerçant, TPE, Admin Lite, Annuaire, aux portails web et aux agents publics.

## Identité principale

Chaque personne possède une identité Mansa unique, indépendante de ses rôles. Une même identité peut être client, propriétaire de commerce, employé, agent public ou administrateur selon les habilitations attribuées.

Identifiants acceptés selon le pays et le canal :

- numéro de téléphone normalisé au format E.164 ;
- adresse électronique vérifiée ;
- identifiant interne opaque ;
- identifiant professionnel ou administratif validé par une organisation partenaire.

Les applications ne doivent jamais utiliser le numéro de téléphone comme clé primaire.

## États d’un compte

- `PENDING_VERIFICATION` : inscription commencée, facteur principal non vérifié ;
- `ACTIVE` : identité utilisable dans les limites de son niveau KYC ;
- `RESTRICTED` : accès partiel, opérations sensibles bloquées ;
- `SUSPENDED` : connexion ou opérations bloquées temporairement ;
- `CLOSED` : compte clôturé, données conservées selon les obligations applicables.

Toute transition d’état doit être motivée, horodatée et auditée.

## Facteurs d’authentification

Le socle supporte :

1. code à usage unique envoyé par canal vérifié ;
2. code secret local ou mot de passe selon l’application ;
3. biométrie de l’appareil comme mécanisme local de déverrouillage, jamais comme preuve serveur autonome ;
4. clé matérielle ou authentificateur fort pour les administrateurs ;
5. certificat ou attestation de terminal pour les TPE et appareils gérés.

Les actions financières et administratives sensibles exigent une authentification renforcée adaptée au risque.

## Sessions

- Les jetons d’accès sont courts et signés.
- Les jetons de renouvellement sont rotatifs, révocables et liés à une session serveur.
- La réutilisation d’un ancien jeton de renouvellement révoque toute la famille de session.
- Chaque session stocke au minimum : identité, appareil, application, environnement, date de création, dernière activité, niveau d’authentification et statut.
- Une déconnexion globale révoque toutes les sessions actives.
- Les secrets de signature ne sont jamais stockés dans le dépôt.

## Appareils et confiance

Un appareil reçoit un identifiant opaque généré par l’application. La confiance peut être augmentée après vérification, mais doit être retirée en cas de changement de sécurité, compromission, réinstallation ou signal de fraude.

Pour les TPE, l’enregistrement associe le terminal, le commerce, le point de vente, l’environnement et les capacités matérielles autorisées.

## Autorisation

Mansa combine :

- RBAC pour les rôles stables ;
- ABAC pour les contraintes de pays, organisation, commerce, point de vente, montant, environnement et niveau KYC ;
- approbation à quatre yeux pour les actions les plus sensibles.

Une autorisation est refusée par défaut. Les interfaces ne doivent pas considérer le masquage d’un bouton comme un contrôle de sécurité.

## Protection contre les abus

- limitation par identité, appareil, adresse réseau, route et partenaire ;
- temporisation progressive des tentatives ;
- détection des changements inhabituels d’appareil ou de localisation ;
- révocation immédiate en cas de risque élevé ;
- messages d’erreur non révélateurs ;
- journalisation sans code OTP, mot de passe, jeton ou donnée biométrique.

## Récupération de compte

La récupération ne doit jamais reposer sur un seul facteur faible. Elle combine les facteurs disponibles, les preuves KYC, les délais de sécurité et, pour les comptes sensibles, une validation humaine tracée.

Un changement de numéro de téléphone déclenche une période de protection configurable pendant laquelle certaines opérations sont limitées.

## Contrats API minimaux

- `POST /v1/auth/challenges`
- `POST /v1/auth/challenges/{id}/verify`
- `POST /v1/auth/sessions`
- `POST /v1/auth/sessions/refresh`
- `DELETE /v1/auth/sessions/{id}`
- `DELETE /v1/auth/sessions`
- `GET /v1/me`
- `GET /v1/me/sessions`
- `POST /v1/devices/enrolments`

Les commandes doivent accepter un identifiant de corrélation. Les opérations de création ou de vérification utilisent une clé d’idempotence lorsque leur répétition peut provoquer un effet externe.

## Critères d’acceptation

- Une identité ne peut pas être créée deux fois avec le même identifiant vérifié dans un même périmètre pays.
- Un OTP expiré, déjà consommé ou dépassant le nombre d’essais est refusé.
- La rotation d’un jeton invalide immédiatement sa version précédente.
- La révocation d’une session empêche tout renouvellement ultérieur.
- Une permission absente est refusée côté serveur.
- Toute suspension, récupération, élévation de privilège et révocation globale génère un événement d’audit.
- Aucun secret d’authentification n’apparaît dans les journaux applicatifs.
