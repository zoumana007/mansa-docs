# Agents publics et habilitations

## 1. Objet

Ce document spécifie la gestion des agents publics habilités à utiliser les services Mansa au nom d’un organisme public. Il complète le catalogue des services publics et correspond au contrat TypeScript `mansa-platform/packages/contracts/src/public-agent-api.ts`.

L’objectif est de garantir qu’une opération sensible — création d’amende, encaissement, décision administrative, émission de carte ou action terrain — puisse toujours être rattachée à un agent identifié, habilité pour un périmètre précis et utilisant, lorsque la politique l’exige, un terminal autorisé.

## 2. Modèle d’agent

Le modèle canonique est `PublicAgent`. Il associe :

- un utilisateur Mansa (`userId`) ;
- un organisme public (`organizationId`) ;
- un matricule métier (`employeeNumber`) ;
- une unité ou un service (`unitCode`) ;
- des rôles (`roleCodes`) ;
- des juridictions (`jurisdictionCodes`) ;
- une liste de terminaux explicitement autorisés (`allowedDeviceIds`) ;
- une période de validité ;
- un statut parmi `INVITED`, `ACTIVE`, `SUSPENDED`, `REVOKED` et `EXPIRED`.

Un même utilisateur peut disposer de plusieurs habilitations uniquement lorsque les politiques de séparation des responsabilités et les accords avec les organismes concernés l’autorisent.

## 3. Routes de référence

Toutes les routes sont sous `/v1/public-services/agents`.

| Opération | Méthode | Route |
| --- | --- | --- |
| Lister les agents | `GET` | `/v1/public-services/agents` |
| Lire un agent | `GET` | `/v1/public-services/agents/:agentId` |
| Enregistrer un agent | `POST` | `/v1/public-services/agents` |
| Modifier les habilitations | `PATCH` | `/v1/public-services/agents/:agentId` |
| Suspendre | `POST` | `/v1/public-services/agents/:agentId/suspension` |
| Réactiver | `POST` | `/v1/public-services/agents/:agentId/reactivation` |
| Révoquer | `POST` | `/v1/public-services/agents/:agentId/revocation` |

Les filtres de liste couvrent au minimum l’organisme, l’utilisateur, le matricule, l’unité, la juridiction et le statut.

## 4. Enregistrement et modification

`RegisterPublicAgentCommand` définit l’identité métier, les rôles, les juridictions, les terminaux autorisés et la période de validité initiale.

`UpdatePublicAgentCommand` permet de modifier uniquement les attributs d’habilitation prévus par le contrat. Une modification de rôles, de juridiction ou de terminaux doit produire un événement d’audit détaillant l’acteur administratif, l’ancienne valeur, la nouvelle valeur, la justification et la corrélation de la demande.

Le matricule ne doit pas être utilisé comme secret ni comme authentifiant unique. L’authentification de l’agent reste celle de l’identité Mansa, renforcée selon le risque de l’opération.

## 5. Suspension, réactivation et révocation

Les changements d’état sensibles utilisent `ChangePublicAgentStatusCommand` avec un motif obligatoire et, si la politique l’exige, un `approvalRequestId`.

- `SUSPENDED` bloque immédiatement les nouvelles opérations tout en conservant l’historique.
- La réactivation n’est possible qu’après contrôle de la validité, des rôles, des juridictions et des terminaux.
- `REVOKED` est un état de retrait définitif de l’habilitation ; une nouvelle habilitation doit être créée si la personne revient ultérieurement.
- `EXPIRED` est déterminé lorsque la date de fin de validité est dépassée.

Aucune suspension ou révocation ne doit supprimer ni altérer les opérations déjà effectuées.

## 6. Contrôles terrain et anti-fraude

Pour une opération terrain comme une amende routière ou un encaissement administratif :

1. l’agent est authentifié ;
2. son statut est `ACTIVE` ;
3. l’organisme correspond au service utilisé ;
4. sa juridiction couvre le lieu ou la catégorie de l’opération ;
5. son rôle autorise l’action demandée ;
6. le terminal figure dans `allowedDeviceIds` lorsque le contrôle terminal est activé ;
7. l’opération produit une trace d’audit corrélée et non modifiable ;
8. le paiement, lorsqu’il existe, génère un reçu vérifiable indépendamment par le citoyen.

Le système ne doit jamais permettre à un agent de modifier un barème officiel localement sur son terminal. Les montants et règles proviennent du catalogue de services configuré et versionné.

## 7. Séparation des responsabilités

Les fonctions suivantes doivent pouvoir être séparées par politique :

- création ou émission d’une obligation ;
- encaissement ;
- annulation ou remboursement ;
- décision de bourse ;
- administration des agents ;
- modification des barèmes ou catalogues ;
- validation d’une opération à risque élevé.

Un administrateur chargé d’enregistrer les agents ne doit pas automatiquement disposer du droit d’encaisser ou d’émettre des obligations.

## 8. Sécurité et données

- Aucun secret de partenaire, code PIN, mot de passe ou jeton d’accès n’est stocké dans le profil agent.
- Les identifiants de terminal sont des références, pas des secrets.
- Les données exportées pour reporting doivent minimiser les informations personnelles.
- Toute consultation transverse entre organismes requiert une permission explicite et est auditée.
- Les changements de rôles ou de juridiction prennent effet de façon déterministe et doivent pouvoir être propagés immédiatement aux contrôles d’autorisation.

## 9. Critères de recette

1. Un agent suspendu ou révoqué ne peut plus effectuer d’opération protégée.
2. Un terminal non autorisé est refusé lorsque la politique de contrôle terminal est active.
3. Une juridiction hors périmètre entraîne un refus explicite et audité.
4. Une modification d’habilitation produit une trace d’audit corrélée.
5. Une révocation ne supprime aucun historique financier ou administratif.
6. Les listes sont paginées via le contrat commun `PageResponse`.
7. Les routes et méthodes correspondent aux constantes `PUBLIC_AGENT_API_ROUTES` et `PUBLIC_AGENT_API_METHODS`.
8. Le contrat est publié via `@mansa/contracts/api-contracts` et le sous-chemin `@mansa/contracts/public-agent-api`.
