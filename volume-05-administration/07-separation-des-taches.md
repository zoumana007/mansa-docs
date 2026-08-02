# Séparation des tâches sensibles

## Objet

La séparation des tâches empêche qu’un même acteur cumule, dans un même périmètre, des rôles permettant d’initier, d’approuver et de contrôler une opération sensible. Elle complète le RBAC, les permissions effectives et les politiques d’autorisation.

## Principe

Une affectation de rôle peut être valide individuellement tout en étant incompatible avec une autre affectation active du même acteur. Le contrôle doit être effectué lors de l’attribution d’un rôle, lors de la réactivation d’une affectation et avant l’exécution d’une action critique.

Le contrôle porte uniquement sur les affectations :

- au statut `ACTIVE` ;
- dont la période de validité couvre l’instant évalué ;
- appartenant au même acteur ;
- appartenant exactement au même périmètre fonctionnel.

Les conflits entre périmètres distincts ne sont pas bloqués automatiquement. Ils doivent néanmoins être analysés par les équipes conformité et sécurité lorsque des relations hiérarchiques existent entre les périmètres.

## Conflits de référence

### Proposition et approbation financières

Les rôles `FINANCE_OPERATOR` et `FINANCE_APPROVER` ne peuvent pas être cumulés par le même acteur dans une même portée. L’auteur d’un ajustement ne doit jamais pouvoir l’approuver lui-même.

### Administration des habilitations et audit

Le rôle `AUDITOR` est incompatible avec `SECURITY_ADMIN` et `COUNTRY_ADMIN` dans une même portée. Un auditeur ne doit pas administrer les droits qu’il est chargé de contrôler.

### Collecte et annulation de paiements publics

Les rôles `PUBLIC_AGENT_COLLECTOR` et `PUBLIC_AGENT_SUPERVISOR` sont incompatibles lorsqu’ils sont affectés explicitement au même acteur dans le même organisme public. L’annulation ou la correction d’un paiement doit être réalisée par une personne distincte.

## Comportement attendu

- Une attribution conflictuelle est refusée avant persistance.
- Le refus expose un code de conflit stable sans divulguer plus d’informations que nécessaire.
- Toute tentative est inscrite dans le journal d’audit.
- Une affectation suspendue, révoquée, future ou expirée ne crée pas de conflit actif.
- La suppression ou la suspension d’une affectation doit faire disparaître le conflit immédiatement.
- Les exceptions temporaires exigent une approbation formelle, une durée limitée et une justification auditable ; elles ne sont pas activées par défaut dans le socle.

## Alignement avec le code

Le contrat de référence est implémenté dans `packages/contracts/src/separation-of-duties.ts` du dépôt `mansa-platform`.

Il expose :

- `DUTY_CONFLICT_RULES`, catalogue des incompatibilités ;
- `findDutyConflicts`, détection détaillée des conflits ;
- `hasDutyConflict`, contrôle synthétique ;
- le sous-chemin `@mansa/contracts/separation-of-duties`.

## Critères d’acceptation

- Le cumul `FINANCE_OPERATOR` + `FINANCE_APPROVER` est détecté dans une même portée.
- Le même cumul dans deux pays différents n’est pas considéré comme un conflit direct.
- Une affectation suspendue ou expirée est ignorée.
- Le cumul `AUDITOR` + `SECURITY_ADMIN` est détecté.
- Le cumul `AUDITOR` + `COUNTRY_ADMIN` est détecté.
- Le cumul `PUBLIC_AGENT_COLLECTOR` + `PUBLIC_AGENT_SUPERVISOR` est détecté dans un même organisme public.
- Le résultat contient le code, l’acteur, la portée, les rôles et les identifiants d’affectation concernés.
- Les tests automatisés couvrent les portées, statuts et dates de validité.
