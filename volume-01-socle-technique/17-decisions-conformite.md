# Décisions de conformité

## Objectif

Une décision de conformité formalise la mesure proposée à l’issue de l’analyse d’un dossier : absence d’action, demande d’informations, restriction de compte, rejet d’une opération ou déclaration de soupçon.

Le contrat technique de référence se trouve dans `mansa-platform/packages/security/src/compliance-decision.ts`.

## Principes

1. Toute décision commence à l’état `PROPOSED`.
2. La proposition conserve son auteur, sa date, son résultat et sa justification.
3. Une approbation ou un rejet exige un acteur différent du proposant.
4. Le proposant peut annuler sa propre proposition avant décision.
5. Une décision finale ne peut pas être décidée une seconde fois.
6. Chaque transition produit une action d’audit stable.
7. Le domaine ne déclenche pas directement les effets externes : il produit une décision que les services applicatifs exécutent de manière contrôlée et idempotente.

## Résultats supportés

- `NO_ACTION` : l’alerte ou le dossier est clôturé sans mesure supplémentaire ;
- `REQUEST_INFORMATION` : des informations ou justificatifs complémentaires sont demandés ;
- `RESTRICT_ACCOUNT` : une restriction métier doit être appliquée au compte ;
- `REJECT_OPERATION` : l’opération concernée doit être bloquée ou rejetée ;
- `REPORT_SUSPICION` : le dossier est orienté vers le processus réglementaire de déclaration.

Les libellés techniques ne remplacent pas les qualifications juridiques à valider avec les partenaires agréés et les autorités compétentes.

## États

- `PROPOSED` : proposition en attente d’une seconde lecture ;
- `APPROVED` : proposition approuvée par un autre acteur habilité ;
- `REJECTED` : proposition refusée avec justification ;
- `CANCELLED` : proposition retirée avant décision.

Les états `APPROVED`, `REJECTED` et `CANCELLED` sont finaux pour l’objet courant. Une nouvelle analyse nécessite une nouvelle décision avec un nouvel identifiant.

## Données minimales

Chaque décision contient :

- un identifiant unique ;
- l’identifiant du dossier conformité ;
- le résultat proposé ;
- une justification non vide ;
- le statut ;
- l’auteur et la date de proposition ;
- après décision, l’acteur décideur, la date et le motif.

## Principe des quatre yeux

Pour `APPROVE` et `REJECT`, l’acteur décideur doit être différent du proposant. Le service applicatif doit aussi vérifier :

- les permissions des deux acteurs ;
- leur périmètre pays et organisation ;
- l’absence de conflit d’intérêts ;
- le niveau d’habilitation requis pour le résultat ;
- l’authentification renforcée en Production lorsque la politique l’impose.

La simple différence d’identifiant ne suffit pas à prouver une séparation organisationnelle correcte ; les contrôles RBAC et de périmètre restent obligatoires.

## Exécution des effets

Après approbation, un orchestrateur applicatif :

1. persiste la décision et son événement d’audit atomiquement ;
2. publie une commande idempotente vers le module concerné ;
3. applique la restriction, le rejet ou la demande d’information ;
4. conserve la référence de l’exécution ;
5. gère les reprises sans doubler l’effet ;
6. alerte l’équipe conformité en cas d’échec partiel.

Une décision `REPORT_SUSPICION` ne doit jamais envoyer directement des données à un tiers depuis le domaine partagé. Elle alimente un workflow réglementaire séparé, protégé et validé juridiquement.

## Confidentialité et audit

- Les justifications ne doivent pas contenir de secrets ni de données inutiles.
- Les informations sensibles sont classifiées et accessibles uniquement aux rôles habilités.
- Les événements `COMPLIANCE_DECISION_PROPOSED`, `COMPLIANCE_DECISION_APPROVED`, `COMPLIANCE_DECISION_REJECTED` et `COMPLIANCE_DECISION_CANCELLED` sont conservés dans le journal d’audit.
- Les exports destinés au support masquent le contenu sensible.
- Toute consultation d’une décision sensible doit être journalisée.

## Critères d’acceptation

- Une proposition avec un champ obligatoire vide est refusée.
- Toute proposition démarre à l’état `PROPOSED`.
- Le proposant ne peut ni approuver ni rejeter sa propre proposition.
- Le proposant peut annuler sa proposition avec un motif.
- Une approbation par un autre acteur passe à `APPROVED`.
- Un rejet par un autre acteur passe à `REJECTED`.
- Une seconde décision sur un état final est refusée.
- Les mutations retournent l’action d’audit, l’acteur et la justification.
- Les tests unitaires couvrent proposition, approbation, annulation et erreurs principales.
- L’export public du package sécurité expose le contrat.
