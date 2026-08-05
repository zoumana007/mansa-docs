# Cycle de vie KYC

## Objet

Ce document définit le cycle de vie minimal d’un dossier de connaissance client (KYC) partagé par les applications Mansa, l’API et les outils d’administration. Il complète les contrats TypeScript de `packages/contracts/src/kyc.ts` et `packages/contracts/src/kyc-api.ts`.

## Statuts

| Statut | Signification | Modifiable par le client |
|---|---|---|
| `DRAFT` | Dossier créé mais non soumis | Oui |
| `SUBMITTED` | Dossier soumis et placé dans la file de traitement | Non, sauf annulation autorisée |
| `IN_REVIEW` | Dossier pris en charge par un contrôleur ou un moteur automatisé | Non |
| `ACTION_REQUIRED` | Informations ou documents complémentaires demandés | Oui |
| `APPROVED` | Dossier accepté et niveau KYC attribué | Non |
| `REJECTED` | Dossier refusé avec motif interne et message client contrôlé | Non |
| `CANCELLED` | Dossier abandonné avant décision finale | Non |

## Transitions autorisées

- `DRAFT` vers `SUBMITTED` ou `CANCELLED`.
- `SUBMITTED` vers `IN_REVIEW` ou `CANCELLED`.
- `IN_REVIEW` vers `ACTION_REQUIRED`, `APPROVED` ou `REJECTED`.
- `ACTION_REQUIRED` vers `SUBMITTED` ou `CANCELLED`.
- `APPROVED`, `REJECTED` et `CANCELLED` sont des états finaux.

Toute autre transition doit être rejetée par le domaine avant toute écriture persistante. Le contrat partagé fournit `canTransitionKycCase`, `assertKycCaseTransition` et `isFinalKycCaseStatus` afin d’appliquer la même règle dans l’API, les workers et les interfaces d’administration.

## Surface API

| Opération | Méthode | Route | Transition ou effet |
|---|---|---|---|
| Créer un brouillon | `POST` | `/v1/kyc/cases` | crée un dossier `DRAFT` |
| Lister les dossiers | `GET` | `/v1/kyc/cases` | lecture paginée et filtrée |
| Consulter un dossier | `GET` | `/v1/kyc/cases/:caseId` | lecture contrôlée par propriétaire ou périmètre administratif |
| Modifier le profil | `PATCH` | `/v1/kyc/cases/:caseId/profile` | autorisé uniquement dans un état modifiable |
| Ajouter un document | `POST` | `/v1/kyc/cases/:caseId/documents` | ajoute une référence de stockage sécurisée |
| Retirer un document | `DELETE` | `/v1/kyc/cases/:caseId/documents/:documentId` | retire une référence avant soumission |
| Soumettre | `POST` | `/v1/kyc/cases/:caseId/submit` | `DRAFT` ou `ACTION_REQUIRED` vers `SUBMITTED` |
| Démarrer la revue | `POST` | `/v1/admin/kyc/cases/:caseId/review/start` | `SUBMITTED` vers `IN_REVIEW` |
| Décider | `POST` | `/v1/admin/kyc/cases/:caseId/review` | approbation, demande d’action ou rejet |
| Annuler | `POST` | `/v1/kyc/cases/:caseId/cancel` | état non final vers `CANCELLED` lorsque permis |

Toutes les écritures portent une clé d’idempotence et une version attendue. La lecture d’un dossier transporte aussi l’identité du demandeur afin que le service puisse vérifier la propriété, le rôle et le périmètre pays avant de répondre.

## Règles de soumission

1. La commande contient une clé d’idempotence et la version attendue du dossier.
2. Tous les documents référencés appartiennent au même utilisateur et au même dossier.
3. Les objets de stockage ont été analysés, chiffrés et classés avant la soumission.
4. Le pays, le programme KYC et les types de documents sont compatibles.
5. Le dossier n’est soumis que depuis `DRAFT` ou après correction depuis `ACTION_REQUIRED`.
6. Une nouvelle soumission incrémente la version et conserve l’historique des demandes précédentes.

## Règles de revue

- La prise en revue est une opération distincte de la décision afin d’empêcher deux contrôleurs de traiter silencieusement le même dossier.
- Le contrôleur ne peut pas être le client concerné.
- Les rôles et périmètres pays doivent être vérifiés avant l’accès au dossier.
- Une approbation doit préciser le niveau KYC résultant.
- Une demande d’action ou un rejet doit contenir un code de motif interne stable.
- Le commentaire interne n’est jamais exposé directement au client.
- Le message client provient d’un catalogue de messages validé et localisable.
- Les décisions sensibles peuvent exiger une double validation selon le niveau de risque.

## Annulation

L’annulation précise l’acteur, son type, la version attendue et un code de motif. Un utilisateur peut uniquement annuler son propre dossier lorsqu’une transition vers `CANCELLED` est autorisée. Une annulation administrative ou système exige une permission explicite et doit rester distinguable dans l’audit.

## Documents et données personnelles

Les contrats ne transportent que des références vers le stockage sécurisé. Les fichiers réels ne sont jamais enregistrés dans Git, les journaux applicatifs ou les événements métier. Les numéros de document sont masqués. Les règles de conservation, suppression, résidence des données et accès doivent être configurées par pays et validées juridiquement avant la production.

## Audit obligatoire

Chaque création, modification, ajout ou retrait de document, soumission, prise en revue, annulation et décision produit un événement d’audit contenant : acteur, rôle, périmètre, dossier, ancienne version, nouvelle version, résultat, date, identifiant de corrélation et motif. Aucun document brut ni secret ne doit apparaître dans l’événement.

## Critères d’acceptation

- Une transition non autorisée échoue sans modifier le dossier.
- Deux commandes avec la même clé d’idempotence produisent le même résultat.
- Une version obsolète retourne une erreur de concurrence explicite.
- Deux prises en revue concurrentes ne peuvent pas attribuer le même dossier à deux contrôleurs.
- Un administrateur hors périmètre pays ne peut ni lire ni décider le dossier.
- Un dossier final ne peut pas revenir dans un état modifiable.
- Les journaux ne contiennent ni document brut, ni numéro complet, ni donnée biométrique.
- La décision, l’annulation et le niveau résultant sont corrélables avec l’événement d’audit.
