# Audit opérationnel des dead-letters outbox

## 1. Objet

Ce document définit la traçabilité obligatoire des opérations manuelles appliquées aux événements outbox ayant épuisé leur budget de tentatives. Il complète `12-outbox-transactionnelle.md` et `13-cablage-runtime-worker-outbox.md`.

Une remise en file manuelle est une action sensible : elle peut provoquer une nouvelle livraison d’un événement métier déjà en échec. Elle doit donc être authentifiée, motivée, corrélée et persistée avant exploitation en production.

## 2. Modèle persistant

Le dépôt `mansa-platform` contient le modèle Prisma `OperationalAuditLog` et sa migration PostgreSQL. Le journal conserve :

- `id` : identifiant UUID de l’enregistrement ;
- `correlationId` : identifiant de corrélation de la requête ;
- `actorId` : identité du service ou opérateur ayant demandé l’action ;
- `actorType` : type d’acteur (`SERVICE_ACCOUNT`, `SYSTEM` ou `USER`) ;
- `action` : action auditée ;
- `resourceType` et `resourceId` : ressource ciblée ;
- `reason` : justification opérationnelle ;
- `metadata` : métadonnées bornées nécessaires au contrôle ;
- `occurredAt` : date serveur de l’action.

Le journal ne doit contenir aucun secret, token, PAN, payload outbox complet ni donnée KYC non nécessaire.

## 3. Remise en file d’une dead-letter

La route interne `POST /v1/internal/ledger/outbox/dead-letters/:eventId/requeue` reste protégée par `InternalServiceGuard`.

En plus du token de service interne, l’appel doit fournir :

- `x-mansa-actor-id` : identité stable du service ou opérateur technique ;
- `x-mansa-operation-reason` : motif explicite de la remise en file ;
- `x-correlation-id` : recommandé ; s’il est absent, l’intercepteur de corrélation en génère un.

Après une remise en file réussie, la plateforme persiste un enregistrement avec l’action `LEDGER_OUTBOX_DEAD_LETTER_REQUEUED`, la ressource `OUTBOX_EVENT`, l’identifiant de l’événement et le seuil `maxAttempts` utilisé.

Aucun audit de succès n’est créé lorsque l’événement n’existe pas ou n’est plus éligible à la remise en file.

## 4. Sécurité

Le mécanisme actuel est une première couche d’audit serveur. Avant exposition à un portail humain, il faudra ajouter :

- authentification utilisateur forte ou identité de workload signée ;
- autorisation RBAC/ABAC dédiée aux opérations outbox ;
- séparation entre consultation et remise en file ;
- double validation configurable pour les environnements sensibles ;
- rétention et export vers un stockage d’audit central ;
- alertes sur répétitions anormales de requeue ;
- politique de minimisation et de conservation.

Le header `x-mansa-actor-id` ne constitue pas à lui seul une preuve d’identité. Sa valeur est digne de confiance uniquement parce que la route est actuellement derrière l’authentification service-to-service. Le passage futur à mTLS ou workload identity devra propager une identité attestée plutôt qu’une simple valeur déclarative.

## 5. Disponibilité et atomicité

La remise en file et l’écriture d’audit sont actuellement deux opérations persistantes successives. L’action métier est réalisée puis l’audit est écrit.

Avant production critique, la plateforme devra soit :

- regrouper la mutation de l’outbox et l’écriture d’audit dans une transaction PostgreSQL unique ;
- soit appliquer un mécanisme équivalent garantissant qu’une remise en file réussie ne puisse pas rester sans trace durable.

Cette exigence est volontairement conservée comme critère de durcissement : l’échec du stockage d’audit ne doit jamais devenir silencieux.

## 6. Critères d’acceptation

Le lot est considéré comme intégré au socle lorsque :

- le schéma et la migration du journal sont présents ;
- une requeue exige un acteur et un motif ;
- l’identifiant de corrélation est conservé ;
- une requeue réussie produit une trace persistante ;
- une requeue refusée n’est pas présentée comme un succès ;
- les tests du contrôleur vérifient l’audit et le rejet des métadonnées manquantes ;
- la documentation opérationnelle reste alignée sur les headers et le nom de l’action.

Le lot n’est pas encore équivalent à un système d’audit de production complet tant que l’atomicité mutation + audit, l’identité attestée et l’export centralisé ne sont pas finalisés.
