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

Une remise en file réussie persiste un enregistrement avec l’action `LEDGER_OUTBOX_DEAD_LETTER_REQUEUED`, la ressource `OUTBOX_EVENT`, l’identifiant de l’événement et le seuil `maxAttempts` utilisé.

Aucun audit de succès n’est créé lorsque l’événement n’existe pas ou n’est plus éligible à la remise en file.

## 4. Sécurité

Le mécanisme actuel apporte une première couche d’audit serveur persistante et atomique avec la mutation concernée. Avant exposition à un portail humain, il faudra encore ajouter :

- authentification utilisateur forte ou identité de workload signée ;
- autorisation RBAC/ABAC dédiée aux opérations outbox ;
- séparation entre consultation et remise en file ;
- double validation configurable pour les environnements sensibles ;
- rétention et export vers un stockage d’audit central ;
- alertes sur répétitions anormales de requeue ;
- politique de minimisation et de conservation.

Le header `x-mansa-actor-id` ne constitue pas à lui seul une preuve d’identité. Sa valeur est digne de confiance uniquement parce que la route est actuellement derrière l’authentification service-to-service. Le passage futur à mTLS ou workload identity devra propager une identité attestée plutôt qu’une simple valeur déclarative.

## 5. Disponibilité et atomicité

La remise en file et l’écriture d’audit sont regroupées dans une transaction PostgreSQL unique via Prisma.

Le comportement attendu est désormais :

1. la transaction est ouverte ;
2. l’événement dead-letter est remis à l’état `PENDING` uniquement s’il reste éligible ;
3. si aucune ligne n’est modifiée, aucun audit n’est créé ;
4. si la mutation réussit, l’enregistrement `OperationalAuditLog` est écrit dans la même transaction ;
5. si l’écriture d’audit échoue, la transaction échoue et la mutation de l’outbox est annulée ;
6. le succès n’est retourné qu’après validation complète de la transaction.

Cette règle empêche qu’une remise en file réussie reste sans trace durable à cause d’un échec d’audit intermédiaire.

## 6. Tests de référence

Le dépôt `mansa-platform` couvre :

- le contrat du contrôleur et les métadonnées d’audit ;
- le rejet des headers opérateur manquants ;
- l’absence d’audit lorsque l’événement n’est plus éligible ;
- l’exécution mutation + audit dans une transaction unique ;
- la propagation d’un échec du stockage d’audit afin de provoquer le rollback transactionnel.

Les tests utilisent des doubles Prisma pour vérifier le contrat d’orchestration. Une validation d’intégration PostgreSQL réelle reste nécessaire avant qualification production afin de couvrir concurrence, isolation et comportement effectif des transactions sous charge.

## 7. Critères d’acceptation

Le lot est considéré comme intégré au socle lorsque :

- le schéma et la migration du journal sont présents ;
- une requeue exige un acteur et un motif ;
- l’identifiant de corrélation est conservé ;
- une requeue réussie produit une trace persistante ;
- mutation et audit sont atomiques ;
- un échec de l’audit fait échouer l’opération complète ;
- une requeue refusée n’est pas présentée comme un succès ;
- les tests du contrôleur et du service vérifient le contrat ;
- la documentation opérationnelle reste alignée sur les headers et le nom de l’action.

Le lot n’est pas encore équivalent à un système d’audit de production complet tant que l’identité attestée, l’export centralisé et les validations PostgreSQL/concurrence ne sont pas finalisés.
