# Catalogue API — identité et sessions

## 1. Portée

Ce catalogue décrit le premier contrat HTTP du domaine identité. Les chemins et méthodes sont déclarés dans `mansa-platform/packages/contracts/src/identity-api.ts` afin d’éviter les divergences entre documentation, API Gateway et applications clientes.

Préfixe : `/v1`.

Toutes les réponses utilisent les enveloppes API communes. Les erreurs suivent le catalogue `ApiErrorCode`. Les opérations de mutation doivent transporter un identifiant de corrélation et, lorsque requis, une clé d’idempotence.

## 2. Routes

### `POST /v1/auth/register`

Crée une identité client et une première session.

Entrée : `RegisterUserCommand`.

Sortie : `AuthenticationResult`.

Contraintes :

- téléphone ou adresse électronique normalisés ;
- consentements et version des conditions enregistrés ;
- aucun mot de passe ni OTP écrit dans les journaux ;
- limitation de débit par appareil, adresse réseau et identifiant ;
- création atomique de l’utilisateur et de la session.

### `POST /v1/auth/sign-in/password`

Authentifie un utilisateur avec son mot de passe.

Entrée : `PasswordSignInCommand`.

Sortie : `AuthenticationResult`.

Le message d’erreur ne doit pas révéler si le compte existe. Un contrôle de risque peut imposer une vérification supplémentaire.

### `POST /v1/auth/otp/request`

Crée un défi OTP pour un objectif explicite.

Entrée : `RequestOtpCommand`.

Sortie : identifiant du défi et date d’expiration.

Le code OTP n’est jamais retourné par l’API hors environnement de démonstration isolé. Le nombre de demandes et de tentatives est limité.

### `POST /v1/auth/otp/verify`

Vérifie un défi OTP et exécute l’objectif associé.

Entrée : `VerifyOtpCommand`.

Sortie : `AuthenticationResult` lorsque la vérification ouvre une session.

Un défi expiré, déjà utilisé ou dépassant le nombre de tentatives doit être refusé.

### `POST /v1/auth/sessions/refresh`

Renouvelle une session à partir d’un jeton de rafraîchissement valide.

Entrée : `RefreshSessionCommand`.

Sortie : `AuthenticationResult` avec rotation du jeton de rafraîchissement.

La réutilisation d’un ancien jeton après rotation entraîne la révocation de la famille de session selon la politique de risque.

### `GET /v1/auth/sessions`

Retourne les sessions et appareils de l’utilisateur courant.

Sortie : liste de `DeviceSession`.

Les secrets, empreintes internes complètes et jetons ne sont jamais exposés.

### `POST /v1/auth/sessions/:sessionId/revoke`

Révoque une session appartenant à l’utilisateur courant ou une session ciblée par un administrateur habilité.

Entrée : `RevokeSessionCommand`.

Sortie : confirmation et date de révocation.

Une révocation est idempotente. Toute révocation administrative est auditée.

### `GET /v1/users/me`

Retourne l’identité de l’utilisateur courant.

Sortie : `UserIdentity`.

La réponse ne contient que les attributs nécessaires au client. Les données KYC sensibles restent dans les routes de conformité dédiées.

## 3. Sécurité commune

- Accès par TLS uniquement en environnement hébergé.
- Jetons d’accès courts et jetons de rafraîchissement rotatifs.
- Validation stricte des entrées avant exécution métier.
- Hachage des mots de passe avec un algorithme adapté et paramètres administrés.
- Protection contre l’énumération de comptes.
- Limitation de débit et délai progressif après échecs.
- Journal d’audit sans secret ni donnée d’authentification brute.
- Révocation possible par utilisateur, support autorisé et moteur de risque.

## 4. Cohérence technique

La source canonique des chemins et méthodes est :

- `IDENTITY_API_ROUTES`
- `IDENTITY_API_METHODS`
- `IdentityApiContract`

Le contrôleur NestJS devra importer ces contrats ou être validé contre eux. Les applications mobiles et web ne doivent pas recopier manuellement les chemins dans plusieurs modules.

## 5. Critères d’acceptation

- Chaque route documentée existe dans `IDENTITY_API_ROUTES` avec la même méthode.
- Les commandes et réponses utilisent les types déjà exportés par `@mansa/contracts`.
- Une session révoquée ne peut plus être rafraîchie.
- Un jeton de rafraîchissement déjà remplacé ne peut pas être réutilisé.
- Une révocation répétée retourne un résultat stable sans créer d’effet secondaire supplémentaire.
- Les listes de sessions ne dévoilent aucun jeton.
- Les erreurs d’authentification ne permettent pas de déterminer l’existence d’un compte.
- Les tentatives et décisions sensibles sont corrélées aux événements d’audit.
