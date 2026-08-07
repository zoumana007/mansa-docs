# Contrat API — Administration

## Objet

Ce document définit le premier contrat partagé entre le portail administrateur, l’API Gateway et les services métier pour les rôles administratifs, les drapeaux de fonctionnalités et les demandes d’approbation.

Le contrat TypeScript correspondant se trouve dans `mansa-platform/packages/contracts/src/administration-api.ts`.

## Principes

- Toutes les routes sont versionnées sous `/v1/admin`.
- Les lectures sont paginées lorsque le volume peut croître.
- Toute modification sensible exige une clé d’idempotence.
- Les décisions d’approbation doivent être auditables et ne doivent jamais supprimer l’historique.
- Les environnements Démo, Recette et Production sont isolés.
- Une personne ne peut pas approuver sa propre demande lorsque la séparation des tâches l’interdit.
- Aucun secret ni valeur de configuration confidentielle ne doit être renvoyé par ces routes.

## Routes initiales

| Fonction | Méthode | Route |
|---|---:|---|
| Lister les rôles | GET | `/v1/admin/roles` |
| Lister les feature flags | GET | `/v1/admin/feature-flags` |
| Lire un feature flag | GET | `/v1/admin/feature-flags/:featureFlagId` |
| Modifier un feature flag | PATCH | `/v1/admin/feature-flags/:featureFlagId` |
| Créer une demande d’approbation | POST | `/v1/admin/approval-requests` |
| Lister les demandes | GET | `/v1/admin/approval-requests` |
| Lire une demande | GET | `/v1/admin/approval-requests/:approvalRequestId` |
| Décider une demande | POST | `/v1/admin/approval-requests/:approvalRequestId/decision` |

## Filtres minimaux

Les rôles peuvent être filtrés par environnement et état actif. Les drapeaux de fonctionnalités peuvent être filtrés par environnement, état d’activation et recherche textuelle. Les demandes d’approbation peuvent être filtrées par statut, demandeur, approbateur, type de ressource et période de création.

## Idempotence

Les opérations `updateFeatureFlag`, `createApprovalRequest` et `decideApprovalRequest` exigent une clé d’idempotence. Une répétition avec la même clé et le même contenu doit produire le même résultat. Une répétition avec la même clé et un contenu différent doit être rejetée.

## Autorisation et double validation

- La lecture des rôles et feature flags exige une permission administrative explicite.
- La modification d’un feature flag de Production peut exiger une authentification renforcée et une approbation secondaire.
- Une décision doit vérifier le rôle, le périmètre, l’environnement, les conflits de séparation des tâches et l’état courant de la demande.
- Une demande déjà approuvée, rejetée, expirée ou annulée ne peut pas recevoir une nouvelle décision incompatible.

## Audit obligatoire

Chaque création, modification ou décision enregistre au minimum : acteur, rôle effectif, environnement, ressource, action, résultat, justification, adresse réseau ou contexte équivalent, identifiant de corrélation, date et clé d’idempotence masquée ou hachée.

## Critères d’acceptation

1. Les noms de routes et méthodes correspondent exactement au contrat TypeScript.
2. Les réponses paginées utilisent le contrat commun `PageResponse`.
3. Une mise à jour sensible sans idempotence est refusée.
4. Un acteur non autorisé reçoit une erreur normalisée sans fuite d’information.
5. Une auto-approbation interdite est bloquée et auditée.
6. Une modification Production ne peut pas être appliquée au périmètre Démo ou Recette par erreur.
7. Les tests couvrent les transitions valides et invalides des demandes d’approbation.

## État d’implémentation

Le contrat est défini dans `packages/contracts/src/administration-api.ts`, réexporté par le catalogue `packages/contracts/src/api-contracts.ts` et publié via le sous-chemin `@mansa/contracts/administration-api`. L’export depuis le barrel racine `packages/contracts/src/index.ts`, puis l’implémentation dans le module d’administration de l’API Gateway avec validation d’entrée, moteur d’autorisation, persistance, audit et tests d’intégration restent à réaliser.
