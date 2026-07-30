# Volume 1 — Sécurité, identité et audit

## 1. Défense en profondeur

La sécurité de Mansa repose sur plusieurs contrôles indépendants : authentification, autorisation, validation métier, limitation de débit, chiffrement, surveillance, audit et procédures humaines. Aucun contrôle unique n’est considéré comme suffisant.

## 2. Authentification

- Sessions courtes avec jetons d’accès signés et rotation des jetons de renouvellement.
- Révocation par appareil, utilisateur, rôle ou incident.
- MFA obligatoire pour les administrateurs et opérations à risque.
- Association des appareils avec possibilité de déconnexion distante.
- Codes à usage unique limités dans le temps, nombre d’essais borné et anti-énumération.
- Réauthentification renforcée avant changement de numéro, ajout de bénéficiaire, modification de limites ou paiement sensible.

## 3. Autorisation

Le RBAC définit les capacités générales, complété par des règles contextuelles : pays, entité, agence, commerce, montant, niveau KYC, horaire et état du compte. Le refus est la valeur par défaut.

Les rôles administratifs sont séparés : support, conformité, risque, finance, opérations, intégrations et super administration. Les opérations critiques utilisent une approbation à quatre yeux.

## 4. Protection des données

- TLS pour toutes les communications.
- Chiffrement au repos des bases, sauvegardes et objets sensibles.
- Masquage des numéros de carte, téléphones et documents dans les interfaces et journaux.
- Aucun PAN complet, CVV, mot de passe ou secret dans les logs.
- Clés et secrets stockés hors GitHub et renouvelés périodiquement.
- Accès aux documents KYC par URL temporaire et autorisation explicite.

## 5. Audit immuable

Chaque action sensible produit un événement contenant : acteur, rôle, organisation, action, ressource, résultat, adresse réseau, appareil, corrélation, justification éventuelle et horodatage. Les événements ne peuvent pas être modifiés par les administrateurs fonctionnels.

Les exports d’audit sont signés ou accompagnés d’une empreinte afin de détecter toute altération.

## 6. Contrôles applicatifs

- Validation stricte des entrées et listes autorisées.
- Protection CSRF pour les sessions web et politique CORS restrictive.
- En-têtes de sécurité et politique de contenu.
- Requêtes paramétrées et ORM sans SQL dynamique non contrôlé.
- Analyse antivirus des fichiers entrants.
- Limitation de débit par route, compte, appareil et adresse réseau.
- Idempotence et protection contre rejeu des webhooks.

## 7. Développement sécurisé

La CI exécute au minimum : lint, vérification TypeScript, tests, build, audit des dépendances, détection de secrets et analyse statique. Les dépendances critiques sont épinglées et les mises à jour sont testées avant fusion.

Aucun environnement de démonstration ne doit contenir de données réelles. Les fonctionnalités simulées sont clairement identifiées et impossibles à confondre avec la production.

## 8. Réponse aux incidents

Chaque incident suit : détection, qualification, confinement, conservation des preuves, correction, communication, rétablissement et retour d’expérience. Les capacités de blocage global, partenaire, compte, carte et fonctionnalité doivent être testées avant production.

## 9. Critères d’acceptation

- Un administrateur ne peut agir hors de son périmètre.
- Une session révoquée ne peut plus être renouvelée.
- Toute opération sensible est retrouvable par identifiant de corrélation.
- Les journaux ne contiennent aucun secret ou numéro de carte complet.
- Les webhooks invalides, expirés ou rejoués sont rejetés.
