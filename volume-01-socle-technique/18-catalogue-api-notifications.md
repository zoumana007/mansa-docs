# Catalogue API — notifications

## 1. Portée

Ce catalogue décrit les routes de création, consultation, annulation et relance des notifications Mansa. Les chemins, méthodes et types partagés sont déclarés dans `mansa-platform/packages/contracts/src/notification-api.ts` et `notification.ts`.

Préfixe : `/v1/notifications`.

Le module doit rester indépendant des fournisseurs SMS, e-mail, push, WhatsApp et messagerie interne. Les applications consomment uniquement le contrat Mansa ; les fournisseurs sont branchés derrière des adaptateurs configurables.

## 2. Envoi

### `POST /v1/notifications`

Crée une demande d’envoi à partir de `SendNotificationCommand` et retourne un `SendNotificationResult` contenant une livraison par canal retenu.

Règles minimales :

- exiger une clé d’idempotence ;
- vérifier l’existence et la version active du modèle `templateKey` ;
- vérifier les variables obligatoires du modèle ;
- appliquer la langue et le fuseau du destinataire ;
- appliquer les préférences de communication, consentements et oppositions ;
- masquer les coordonnées dans les réponses et journaux ;
- refuser les canaux non autorisés pour le type de message ;
- ne jamais utiliser une notification comme preuve unique d’une transaction financière ;
- enregistrer un identifiant de corrélation pour les opérations critiques ;
- programmer l’envoi lorsque `scheduledAt` est fourni.

Une commande multicanale produit plusieurs `NotificationDelivery`. L’échec d’un canal ne doit pas masquer l’état des autres canaux.

## 3. Consultation

### `GET /v1/notifications/deliveries`

Liste les livraisons accessibles avec pagination et filtres :

- `userId` ;
- `templateKey` ;
- `channel` ;
- `status` ;
- `correlationId` ;
- `createdFrom` ;
- `createdTo`.

Les utilisateurs finaux ne consultent que leurs propres notifications. Les équipes support et exploitation accèdent aux métadonnées nécessaires selon leurs permissions, sans voir les secrets, contenus sensibles ou coordonnées complètes.

### `GET /v1/notifications/deliveries/:deliveryId`

Retourne une livraison autorisée. La réponse expose le destinataire masqué, le canal, le statut, les références techniques non sensibles et les dates de traitement.

Une livraison hors portée ne doit pas révéler son existence.

## 4. Annulation et relance

### `POST /v1/notifications/deliveries/:deliveryId/cancel`

Annule une livraison encore annulable avec `NotificationDeliveryActionCommand`.

Règles minimales :

- vérifier la concordance entre l’identifiant du chemin et celui de la commande ;
- exiger une clé d’idempotence ;
- exiger un motif ;
- autoriser uniquement les statuts `PENDING` ou `QUEUED` ;
- tracer l’acteur, le motif et l’horodatage ;
- propager l’annulation au fournisseur lorsque celui-ci le permet.

### `POST /v1/notifications/deliveries/:deliveryId/retry`

Relance une livraison échouée avec `NotificationDeliveryActionCommand`.

Règles minimales :

- limiter la relance aux statuts et erreurs éligibles ;
- ne pas relancer une erreur définitive ou un destinataire opposé ;
- appliquer une temporisation et un nombre maximal de tentatives ;
- créer une nouvelle tentative corrélée à la livraison initiale ;
- empêcher les doubles relances avec la clé d’idempotence ;
- auditer les relances manuelles.

## 5. Cycle de vie

Les statuts partagés sont :

- `PENDING` : demande acceptée mais pas encore mise en file ;
- `QUEUED` : demande placée dans la file de traitement ;
- `SENT` : remise au fournisseur ;
- `DELIVERED` : livraison confirmée ;
- `FAILED` : échec temporaire ou définitif ;
- `CANCELLED` : envoi annulé avant livraison.

Les transitions doivent être contrôlées. Un statut final ne peut être remplacé que par une opération explicitement définie, comme une nouvelle tentative séparée.

## 6. Sécurité, confidentialité et conformité

- Aucun mot de passe, code PIN, secret complet, PAN ou document KYC ne doit être inséré dans une notification.
- Les codes OTP doivent être courts, expirables, à usage unique et exclus des journaux.
- Les coordonnées sont chiffrées au repos et masquées dans les interfaces.
- Les modèles sont versionnés, validés et séparés par environnement.
- Les messages transactionnels et marketing suivent des règles de consentement distinctes.
- Les liens sensibles sont signés, expirables et liés à une action précise.
- Les webhooks fournisseurs sont authentifiés, dédupliqués et corrélés.
- Les quotas protègent contre le spam, les boucles et les coûts anormaux.
- Les erreurs fournisseur ne doivent pas exposer de données internes au client.

## 7. Résilience et exploitation

- Files durables avec reprise après incident.
- Stratégie de retry avec backoff et jitter.
- File de quarantaine pour les erreurs non récupérables.
- Adaptateur de secours configurable par canal et pays.
- Métriques de débit, latence, livraison, échec et coût.
- Alertes sur hausse d’échecs, retard de file, consommation anormale et indisponibilité fournisseur.
- Réconciliation périodique entre statuts internes et accusés fournisseurs.

## 8. Cohérence technique

La source canonique est constituée de :

- `NOTIFICATION_API_ROUTES` ;
- `NOTIFICATION_API_METHODS` ;
- `NotificationApiContract` ;
- `ListNotificationDeliveriesQuery` ;
- `NotificationDeliveryActionCommand` ;
- `SendNotificationResult` ;
- les types du fichier `notification.ts`.

Le paquet `@mansa/contracts` expose ce catalogue via `@mansa/contracts/notification-api`. Les contrôleurs NestJS, workers et applications doivent importer ces contrats au lieu de redéfinir les chemins ou charges utiles.

## 9. Critères d’acceptation

- Chaque route documentée possède la même méthode et le même chemin dans le catalogue TypeScript.
- Une même clé d’idempotence ne crée pas plusieurs envois logiques.
- Une commande multicanale retourne une livraison distincte par canal.
- Les préférences, consentements et oppositions sont respectés.
- Les coordonnées sont masquées et les secrets absents des contenus et journaux.
- Les annulations et relances suivent des transitions contrôlées et auditées.
- Les erreurs temporaires sont relancées selon une politique bornée.
- Les accusés fournisseurs sont authentifiés et dédupliqués.
- Les métriques permettent de détecter rapidement retard, échec ou surcoût.
