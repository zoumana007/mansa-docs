# Décisions d’autorisation

## Objet

Ce document définit le contrat commun utilisé par l’API Gateway, les services métier, l’administration et l’audit pour expliquer une décision d’accès. Une décision ne se limite pas à `autorisé` ou `refusé` : elle porte un motif stable, les obligations à appliquer et la liste des politiques évaluées.

## Principe de refus par défaut

Toute action est refusée lorsqu’aucune règle active ne l’autorise explicitement. Une permission de rôle constitue seulement une condition nécessaire. Le moteur vérifie également le type d’acteur, le périmètre, le niveau d’authentification, l’état de la ressource, l’environnement, les limites et les politiques de refus prioritaires.

## Codes d’autorisation

Les décisions positives utilisent uniquement :

- `ALLOWED_BY_POLICY` : une politique active autorise explicitement l’action ;
- `ALLOWED_BY_ROLE` : le profil de rôle et le périmètre suffisent pour une action non sensible.

Les refus utilisent notamment :

- `DENIED_BY_POLICY` ;
- `DENIED_MISSING_PERMISSION` ;
- `DENIED_ACTOR_TYPE` ;
- `DENIED_SCOPE` ;
- `DENIED_AUTHENTICATION_LEVEL` ;
- `DENIED_RESOURCE_STATE` ;
- `DENIED_AMOUNT_LIMIT` ;
- `DENIED_ENVIRONMENT` ;
- `DENIED_POLICY_NOT_FOUND` ;
- `DENIED_POLICY_INACTIVE` ;
- `DENIED_DEFAULT`.

Les applications peuvent afficher un message adapté, mais elles ne doivent jamais remplacer le code technique ni révéler une information sensible dans le message utilisateur.

## Obligations

Une autorisation peut imposer une ou plusieurs obligations avant ou après l’action :

- authentification multifacteur ou liée au matériel ;
- authentification renforcée ponctuelle ;
- double approbation ;
- KYC récent ;
- revue risque ;
- création d’un événement d’audit ;
- masquage des champs sensibles ;
- limitation aux ressources propres ou au périmètre affecté ;
- interdiction d’export.

Une obligation non satisfaite transforme l’exécution en refus contrôlé. Elle ne doit pas être ignorée par un service consommateur.

## Ordre minimal d’évaluation

1. Valider l’identité et la session.
2. Vérifier le type d’acteur.
3. Vérifier la permission demandée.
4. Vérifier le périmètre affecté et la ressource cible.
5. Vérifier le niveau d’authentification.
6. Appliquer les politiques de refus prioritaires.
7. Vérifier les limites, l’environnement et l’état de la ressource.
8. Produire les obligations et la trace d’évaluation.
9. Journaliser les décisions sensibles et tous les refus administratifs.

## Traçabilité

Chaque décision sensible doit être corrélée à la requête d’origine. La trace contient au minimum : l’acteur, l’action, la ressource, le code de décision, les politiques évaluées, les obligations, l’horodatage et le `correlationId`.

Les données personnelles et secrets ne sont pas copiés dans le journal. Les exports d’audit appliquent les règles de masquage et de rétention.

## Alignement avec le code

Le contrat technique se trouve dans `packages/contracts/src/authorization-decision.ts` de `mansa-platform`. Le paquet `@mansa/contracts` l’expose aussi par le sous-chemin `@mansa/contracts/authorization-decision`.

Les constantes `AUTHORIZATION_DECISION_REASON_CODES` et `AUTHORIZATION_OBLIGATIONS` constituent la source de vérité partagée. Toute nouvelle valeur exige une mise à jour de ce document, des tests, de l’API et des tableaux de bord concernés.

## Critères d’acceptation

- Les codes et obligations sont uniques et versionnés.
- Un code inconnu est rejeté par les gardes de type.
- Un refus est audité par défaut.
- Une décision autorisée ne supprime pas les obligations.
- Les politiques évaluées restent traçables par identifiant.
- Les applications n’accordent jamais un accès en interprétant uniquement un libellé utilisateur.
- Les services refusent l’exécution lorsqu’une obligation obligatoire n’est pas satisfaite.
