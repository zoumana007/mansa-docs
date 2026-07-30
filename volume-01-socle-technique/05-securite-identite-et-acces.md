# Volume 1 — Sécurité, identité et contrôle d’accès

## 1. Objectif

La sécurité de Mansa repose sur une défense en profondeur. Aucun contrôle isolé ne doit être considéré comme suffisant pour protéger les comptes, les données personnelles ou les opérations financières.

## 2. Authentification

- Identifiant principal configurable par pays : téléphone, e-mail ou identifiant institutionnel.
- Mot de passe ou code secret stocké uniquement sous forme de dérivé robuste avec sel.
- Jetons d’accès courts et jetons de renouvellement rotatifs.
- Révocation de session disponible depuis le client et l’administration.
- Authentification renforcée obligatoire pour les actions sensibles : ajout de bénéficiaire, changement d’appareil, transfert important, modification KYC, changement de limites et actions administratives critiques.
- Liaison d’appareil avec empreinte non intrusive, clé locale et historique vérifiable.

## 3. Autorisation

Mansa combine :

- RBAC pour les rôles stables ;
- règles contextuelles pour le pays, l’organisation, le point de vente, le niveau KYC, le canal et le montant ;
- séparation des responsabilités pour les opérations à risque ;
- double validation configurable pour remboursement, déblocage, modification de commission et export sensible.

Un refus d’autorisation est la valeur par défaut lorsque la règle applicable est absente ou ambiguë.

## 4. Rôles de référence

- `CUSTOMER` : particulier.
- `MERCHANT_OWNER` : propriétaire ou responsable de commerce.
- `MERCHANT_EMPLOYEE` : employé limité à un ou plusieurs points de vente.
- `FIELD_AGENT` : agent terrain.
- `PUBLIC_AGENT` : agent d’une administration partenaire.
- `SUPPORT_AGENT` : support avec données masquées.
- `COMPLIANCE_ANALYST` : conformité et revue KYC.
- `RISK_ANALYST` : risque et fraude.
- `FINANCE_OPERATOR` : rapprochements et exports financiers.
- `TENANT_ADMIN` : administrateur d’une organisation.
- `SUPER_ADMIN` : administration globale soumise à des contrôles renforcés.

## 5. Journal d’audit

Toute action critique enregistre : acteur, rôle, organisation, cible, type d’action, résultat, horodatage, adresse réseau, appareil, motif et identifiant de corrélation. Les valeurs sensibles sont masquées. Les journaux sont append-only et exportables vers un stockage immuable.

## 6. Protection des données

- Chiffrement TLS en transit.
- Chiffrement au repos pour bases, sauvegardes et stockage objet.
- Données KYC séparées logiquement des données transactionnelles.
- Masquage des numéros de téléphone, cartes, documents et comptes dans les interfaces non autorisées.
- Durées de conservation configurées selon le pays et la nature de la donnée.
- Suppression ou anonymisation uniquement lorsque les obligations légales et comptables le permettent.

## 7. Gestion des secrets

Aucun secret réel n’est versionné. Les secrets de production proviennent d’un gestionnaire dédié, sont limités par environnement, soumis à rotation et accessibles au strict minimum. Les fichiers `.env` locaux ne sont jamais committés.

## 8. Sécurité applicative

- Validation stricte des entrées.
- Limitation de débit par adresse, appareil, compte et route.
- Protection contre rejeu avec idempotence, nonce et fenêtre temporelle.
- En-têtes de sécurité et politique CORS explicite.
- Requêtes paramétrées via l’ORM.
- Analyse automatisée des dépendances et détection de secrets en CI.
- Tests d’autorisation sur chaque route sensible.

## 9. Réponse aux incidents

Le système doit permettre de désactiver immédiatement : un compte, une session, un appareil, une clé API, un partenaire, un canal, une route, un pays ou une fonctionnalité. Chaque incident possède une chronologie, un responsable, les mesures prises et un rapport de clôture.

## 10. Critères d’acceptation

- Aucun endpoint métier sensible sans authentification et règle d’autorisation explicite.
- Rotation des jetons de renouvellement testée.
- Révocation globale des sessions testée.
- Journaux d’audit générés pour toutes les mutations administratives.
- Données sensibles absentes des logs techniques.
- Contrôles de sécurité automatisés exécutés dans la CI.
