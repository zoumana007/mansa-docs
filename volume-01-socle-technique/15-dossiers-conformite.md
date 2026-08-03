# Dossiers conformité

## Objectif

Les dossiers conformité structurent le traitement humain des alertes issues de la surveillance transactionnelle, du filtrage de sanctions ou d’un signalement manuel. Ils garantissent qu’une alerte ne peut pas être supprimée, résolue ou fermée sans état, responsable, justification et trace d’audit.

Le contrat technique de référence se trouve dans `mansa-platform/packages/security/src/compliance-case.ts`.

## Sources

Un dossier peut être créé depuis :

- `TRANSACTION_MONITORING` : décision `REVIEW` ou `BLOCK` du moteur de surveillance ;
- `SCREENING` : correspondance potentielle avec une liste de sanctions, PPE ou autre référentiel ;
- `MANUAL` : signalement documenté par un agent habilité.

Chaque dossier conserve une référence vers la source d’origine sans recopier inutilement les données sensibles.

## États

- `OPEN` : dossier créé et en attente de prise en charge ;
- `IN_REVIEW` : analyse active par un agent assigné ;
- `ESCALATED` : risque élevé nécessitant une équipe ou autorité supérieure ;
- `RESOLVED` : décision motivée enregistrée, en attente de clôture administrative ;
- `CLOSED` : dossier clôturé, conservé selon la politique de rétention.

## Transitions autorisées

1. Un dossier peut être assigné tant qu’il n’est pas fermé.
2. Une revue ne peut commencer qu’après assignation.
3. Un dossier `OPEN` ou `IN_REVIEW` peut être escaladé avec justification.
4. Une escalade positionne la priorité à `CRITICAL` dans le premier socle.
5. Seul un dossier `IN_REVIEW` ou `ESCALATED` peut être résolu.
6. Une résolution exige un code et une justification non vide.
7. Seul un dossier `RESOLVED` peut être fermé.
8. Un dossier `RESOLVED` ou `CLOSED` peut être rouvert avec justification.
9. Une réouverture efface le code de résolution courant mais conserve l’historique d’audit.
10. Un dossier fermé ne peut pas être modifié sans réouverture explicite.

## Audit

Chaque commande produit une action d’audit normalisée :

- `COMPLIANCE_CASE_ASSIGNED` ;
- `COMPLIANCE_CASE_REVIEW_STARTED` ;
- `COMPLIANCE_CASE_ESCALATED` ;
- `COMPLIANCE_CASE_RESOLVED` ;
- `COMPLIANCE_CASE_CLOSED` ;
- `COMPLIANCE_CASE_REOPENED`.

L’événement persistant doit également contenir l’identifiant du dossier, l’acteur, l’horodatage, l’ancien état, le nouvel état, la justification éventuelle et un identifiant de corrélation. Les justificatifs sensibles restent dans un stockage protégé et ne sont pas inclus dans les journaux techniques.

## Autorisations

- Les agents conformité peuvent consulter et traiter les dossiers de leur périmètre.
- Les analystes risque peuvent consulter les signaux et escalader selon leur rôle.
- Le support général ne voit pas les détails sensibles.
- La clôture, la réouverture et les décisions à fort impact peuvent exiger une double validation en Production.
- Un agent ne peut pas contourner les contrôles de périmètre pays, organisation ou partenaire.

## Intégration applicative

Le service applicatif :

1. crée ou retrouve le dossier de manière idempotente depuis la référence source ;
2. charge la version courante ;
3. vérifie RBAC, périmètre et séparation des tâches ;
4. appelle `transitionComplianceCase` ;
5. persiste l’état et l’événement d’audit dans une transaction atomique ;
6. publie un événement métier après commit ;
7. met à jour les délais de traitement et tableaux de bord ;
8. notifie uniquement les acteurs autorisés.

## Données minimales

- identifiant du dossier ;
- sujet concerné ;
- type et référence de source ;
- état et priorité ;
- propriétaire éventuel ;
- code de résolution éventuel ;
- dates de création et mise à jour ;
- échéance de traitement ;
- historique append-only des transitions.

Aucune donnée KYC réelle, pièce d’identité, clé d’API ou secret partenaire ne doit être stocké dans les dépôts.

## Critères d’acceptation

- La revue d’un dossier non assigné est refusée.
- L’escalade sans justification est refusée.
- La résolution avant revue est refusée.
- La fermeture d’un dossier non résolu est refusée.
- La réouverture d’un dossier résolu ou fermé rétablit l’état `OPEN`.
- Toutes les transitions produisent une action d’audit stable.
- Les tests unitaires couvrent le parcours nominal et les transitions interdites.
- La documentation et les exports du package sécurité correspondent au contrat TypeScript.
