# Catalogue API — services publics

## 1. Portée

Ce catalogue décrit les routes du module État : organisations publiques, catalogue de services, obligations à payer, encaissements, reçus, bourses et cartes étudiantes. Les chemins, méthodes et types partagés sont déclarés dans `mansa-platform/packages/contracts/src/public-services-api.ts` et `public-services.ts`.

Préfixe : `/v1/public-services`.

Les routes doivent respecter la séparation des organisations, juridictions, pays, environnements et rôles. Toute émission, décision ou collecte sensible est auditée et soumise aux politiques d’autorisation.

## 2. Organisations et catalogue

### `GET /v1/public-services/organizations`

Liste les organisations accessibles avec pagination et filtres `countryCode`, `administrativeAreaCode` et `active`.

Les résultats sont limités par la portée de l’acteur. Les comptes de règlement et informations internes non nécessaires ne doivent pas être exposés aux clients publics.

### `GET /v1/public-services/catalog`

Liste les services activés avec filtres `organizationId`, `obligationType` et `active`.

Chaque entrée de catalogue est versionnée. Le montant, la devise, les canaux de paiement, les dates d’effet et les règles de paiement partiel doivent provenir de la version enregistrée au moment de l’émission.

## 3. Obligations et paiements publics

### `POST /v1/public-services/obligations`

Crée une obligation depuis `CreatePublicObligationCommand` et retourne un `PublicObligation`.

Règles minimales :

- exiger une clé d’idempotence ;
- vérifier l’organisation, le service, sa version et sa période d’effet ;
- vérifier que le montant respecte la configuration du service ;
- limiter l’agent à sa juridiction et à ses rôles ;
- générer une référence unique et non prédictible ;
- ne jamais stocker plus de données personnelles que nécessaire ;
- auditer l’émission, l’appareil, l’agent et la justification.

### `GET /v1/public-services/obligations`

Liste les obligations accessibles selon `ListPublicObligationsQuery` avec filtres organisation, service, sujet, référence externe et statut.

### `GET /v1/public-services/obligations/:obligationId`

Retourne une obligation autorisée. Une obligation hors portée ne doit pas révéler son existence.

### `POST /v1/public-services/obligations/:obligationId/payments`

Collecte un paiement avec `CollectPublicPaymentCommand` et retourne un `PublicPaymentReceipt`.

Règles minimales :

- vérifier la concordance entre l’identifiant du chemin et celui de la commande ;
- exiger et dédupliquer la clé d’idempotence ;
- vérifier le statut, le solde restant, la devise et la politique de paiement partiel ;
- vérifier le canal, l’agent collecteur et le terminal lorsque fournis ;
- créer les écritures du ledger avant de confirmer le paiement ;
- appliquer frais et règles de règlement configurés ;
- produire un reçu vérifiable sans exposer de donnée sensible ;
- empêcher tout double encaissement.

### `GET /v1/public-services/receipts/:receiptId`

Retourne un reçu autorisé. Le code de vérification doit permettre un contrôle d’authenticité sans donner accès aux données privées du payeur.

## 4. Bourses

### `GET /v1/public-services/scholarships`

Liste les demandes selon organisation, programme, année académique, bénéficiaire et statut.

### `POST /v1/public-services/scholarships/:applicationId/decision`

Décide une demande avec `DecideScholarshipCommand` et retourne la demande mise à jour.

Règles minimales :

- décision réservée aux rôles autorisés ;
- séparation entre instruction, décision et paiement ;
- double validation selon seuil et politique ;
- montant approuvé obligatoire pour une approbation ;
- justification obligatoire ;
- transition de statut contrôlée et auditée ;
- aucun paiement automatique avant validation du bénéficiaire et du compte de destination.

## 5. Cartes étudiantes

### `POST /v1/public-services/student-cards`

Émet une carte avec `IssueStudentCardCommand` et retourne un `StudentCard`.

Règles minimales :

- exiger une clé d’idempotence ;
- vérifier l’établissement, l’étudiant, l’année et les droits ;
- garantir l’unicité de la carte active pour la même période selon la politique ;
- produire un identifiant public distinct des identifiants internes ;
- tracer remplacement, suspension, révocation et expiration.

### `GET /v1/public-services/student-cards`

Liste les cartes selon organisation, utilisateur, référence étudiante, année et statut.

### `GET /v1/public-services/student-cards/:cardId`

Retourne une carte autorisée avec ses droits non sensibles et son état courant.

## 6. Sécurité et anti-corruption

- Identité forte pour les agents et liaison aux appareils autorisés.
- Portée limitée par organisation, unité et juridiction.
- Séparation des tâches pour émission, annulation, décision, remboursement et règlement.
- Double validation pour les actions à risque ou dépassant un seuil.
- Journal d’audit immuable avec acteur, appareil, localisation déclarée, motif et corrélation.
- Reçus signés ou vérifiables et rapprochement quotidien.
- Alertes sur annulations, doublons, volumes anormaux et contournements de seuils.
- Aucun secret, document réel, numéro d’identité complet ou donnée de carte dans les journaux.
- Fonction de suspension immédiate d’un agent, appareil, service ou organisation.

## 7. Cohérence technique

La source canonique est constituée de :

- `PUBLIC_SERVICES_API_ROUTES`
- `PUBLIC_SERVICES_API_METHODS`
- `PublicServicesApiContract`
- les requêtes de liste du fichier `public-services-api.ts`
- les commandes, statuts et ressources du fichier `public-services.ts`

Le paquet `@mansa/contracts` expose ce catalogue via `@mansa/contracts/public-services-api`. Les contrôleurs NestJS et les applications concernées doivent importer ces contrats au lieu de redéfinir les chemins ou charges utiles.

## 8. Critères d’acceptation

- Chaque route documentée possède la même méthode et le même chemin dans le catalogue TypeScript.
- Toute émission et tout paiement public sont idempotents.
- Un agent ne peut agir qu’au sein de sa juridiction et de ses permissions.
- Les montants et canaux proviennent d’une version active du catalogue.
- Le paiement crée des écritures équilibrées et un reçu vérifiable.
- Une bourse respecte la séparation instruction, décision et paiement.
- Une carte étudiante possède un cycle de vie contrôlé et audité.
- Les données sensibles ne sont ni exposées ni journalisées.
