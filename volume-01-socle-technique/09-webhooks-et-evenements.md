# Webhooks et événements partenaires

## 1. Objectif

Le sous-système de webhooks permet à Mansa de notifier de façon fiable les banques, opérateurs Mobile Money, commerçants, administrations et autres partenaires lorsqu’un événement métier autorisé se produit.

Il ne remplace pas le bus d’événements interne. Les événements internes servent au découplage entre modules Mansa ; les webhooks servent uniquement aux échanges sortants vers des systèmes partenaires.

## 2. Principes obligatoires

- Chaque événement possède un identifiant unique, un type stable, une date d’occurrence, un identifiant de corrélation et une version de schéma.
- Chaque livraison possède son propre identifiant et son propre cycle de vie.
- Une même livraison peut être retentée sans produire plusieurs effets chez le destinataire.
- Le corps signé ne contient jamais de secret, de code OTP, de numéro de carte complet ni de document KYC.
- L’URL de destination doit utiliser HTTPS hors environnements locaux contrôlés.
- Les clés de signature sont stockées dans un gestionnaire de secrets ; seul leur identifiant de référence est conservé dans les données métier.
- Les réponses, erreurs, durées et codes HTTP sont journalisés sans exposer de données sensibles.
- Une suspension administrative d’abonnement doit prendre effet immédiatement.

## 3. Cycle de vie d’un abonnement

Un abonnement peut être :

- `ACTIVE` : les événements compatibles peuvent être livrés ;
- `PAUSED` : aucune nouvelle livraison n’est démarrée, mais l’historique reste consultable ;
- `DISABLED` : l’abonnement est désactivé durablement et nécessite une action explicite pour être réactivé.

La création exige une clé d’idempotence et la liste explicite des types d’événements autorisés. Une modification de l’URL, du statut ou des événements souscrits doit être auditée.

## 4. Cycle de vie d’une livraison

Les statuts sont :

1. `PENDING` : livraison créée mais non commencée ;
2. `DELIVERING` : tentative en cours ;
3. `DELIVERED` : réponse considérée comme réussie ;
4. `RETRY_SCHEDULED` : nouvelle tentative planifiée ;
5. `FAILED` : politique de tentatives épuisée ou erreur non retentable ;
6. `CANCELLED` : livraison annulée avant exécution irréversible.

Une relance manuelle exige une justification, une clé d’idempotence et une autorisation administrative adaptée.

## 5. Signature et vérification

Chaque requête sortante doit au minimum contenir :

- l’identifiant de l’événement ;
- l’identifiant de la livraison ;
- le type d’événement ;
- la date de signature ;
- la version de schéma ;
- une signature HMAC calculée sur le corps brut et les métadonnées retenues par la convention technique.

Le partenaire doit vérifier la signature, rejeter les dates trop anciennes et mémoriser les identifiants déjà traités afin de garantir l’idempotence côté réception.

La rotation des clés doit permettre une période de chevauchement contrôlée sans interruption des livraisons.

## 6. Politique de nouvelle tentative

La politique exacte reste configurable par environnement et partenaire, mais doit respecter les règles suivantes :

- temporisation croissante avec jitter ;
- nombre maximal de tentatives ;
- aucune relance automatique pour les erreurs explicitement non retentables ;
- mise en quarantaine après épuisement ;
- alerte lorsqu’un taux d’échec dépasse le seuil configuré ;
- possibilité de relancer une livraison isolée après correction du système partenaire.

Les codes `408`, `429` et les erreurs serveur peuvent être considérés comme retentables selon la politique. Les erreurs fonctionnelles permanentes doivent être classées sans boucle infinie.

## 7. Contrat API partagé

Le dépôt plateforme expose les types dans :

- `packages/contracts/src/webhook.ts` pour les abonnements, événements, livraisons et statuts ;
- `packages/contracts/src/webhook-api.ts` pour les routes, méthodes, filtres et contrats de transport.

Routes initiales :

- `POST /v1/webhooks/subscriptions` ;
- `GET /v1/webhooks/subscriptions` ;
- `GET /v1/webhooks/subscriptions/:subscriptionId` ;
- `PATCH /v1/webhooks/subscriptions/:subscriptionId` ;
- `GET /v1/webhooks/deliveries` ;
- `GET /v1/webhooks/deliveries/:deliveryId` ;
- `POST /v1/webhooks/deliveries/:deliveryId/retry`.

Les listes doivent être paginées et filtrables. Les réponses d’erreur suivent les conventions communes du dépôt plateforme.

## 8. Sécurité et autorisations

- La gestion des abonnements est limitée aux propriétaires autorisés et aux administrateurs habilités.
- La consultation des livraisons doit respecter le périmètre organisationnel.
- Les clés de signature ne sont jamais retournées en clair après création.
- Les URL privées, locales, de métadonnées cloud ou non autorisées doivent être bloquées afin de réduire le risque SSRF.
- Les redirections HTTP sont désactivées par défaut ou strictement contrôlées.
- Les payloads sont limités en taille et validés avant émission.
- Toute action manuelle de relance ou de changement de statut est auditée.

## 9. Observabilité

Les métriques minimales sont :

- nombre de livraisons par statut et partenaire ;
- taux de succès ;
- latence de livraison ;
- nombre moyen de tentatives ;
- âge de la plus ancienne livraison en attente ;
- nombre de livraisons en échec définitif ;
- volume par type d’événement.

Les journaux doivent conserver les identifiants de corrélation, d’événement, de livraison et d’abonnement sans conserver le secret de signature.

## 10. Critères d’acceptation

- Une création répétée avec la même clé d’idempotence ne crée pas plusieurs abonnements.
- Un abonnement suspendu ne reçoit aucune nouvelle livraison.
- Une livraison réussie n’est pas relancée automatiquement.
- Une erreur retentable planifie une nouvelle tentative conformément à la politique.
- Une erreur permanente atteint `FAILED` sans boucle infinie.
- Une relance manuelle répétée avec la même clé d’idempotence ne crée qu’une seule action.
- Les utilisateurs hors périmètre ne peuvent ni consulter ni modifier l’abonnement.
- Une URL interdite par la politique SSRF est rejetée.
- La signature peut être vérifiée avec la clé active correspondante.
- Les événements et livraisons restent corrélables de bout en bout dans les logs et métriques.
