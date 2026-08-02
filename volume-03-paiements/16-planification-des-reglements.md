# Planification des règlements

## Objet

La planification détermine quand les encaissements éligibles d’un commerçant sont regroupés en lots de règlement. Elle complète le cycle décrit dans `15-reglement-des-commercants.md` sans exécuter elle-même le paiement.

## Fréquences prises en charge

- `INSTANT` : déclenchement après chaque opération devenue éligible, sous réserve des contrôles et du seuil minimal ;
- `DAILY` : constitution quotidienne après l’heure de coupure ;
- `WEEKLY` : constitution un jour défini de la semaine ;
- `MONTHLY` : constitution un jour défini du mois ;
- `MANUAL` : déclenchement autorisé uniquement par une action administrative auditée.

## États

- `ACTIVE` : la planification peut générer des lots ;
- `PAUSED` : aucun nouveau lot n’est généré, sans supprimer la configuration ;
- `DISABLED` : configuration désactivée administrativement.

La suspension ou la désactivation n’annule jamais un lot déjà créé.

## Paramètres

Chaque planification contient :

- un identifiant unique ;
- un commerçant ;
- une fréquence ;
- un fuseau horaire métier ;
- une heure de coupure UTC facultative ;
- un jour de semaine pour `WEEKLY` ;
- un jour du mois pour `MONTHLY` ;
- un seuil minimal exprimé en unité monétaire mineure ;
- une devise ;
- un état et des horodatages.

Le jour de semaine utilise les valeurs 1 à 7. Le jour du mois est limité à 1–28 afin d’éviter les dates inexistantes et les différences de longueur entre les mois.

## Règles métier

1. Une planification appartient à un seul commerçant et une seule devise.
2. Les montants sont des entiers sûrs non négatifs.
3. La devise est normalisée sur trois lettres majuscules.
4. Une fréquence hebdomadaire exige `dayOfWeek` et interdit `dayOfMonth`.
5. Une fréquence mensuelle exige `dayOfMonth` et interdit `dayOfWeek`.
6. Les autres fréquences interdisent ces deux paramètres.
7. L’heure de coupure, lorsqu’elle est fournie, est comprise entre 0 et 23.
8. Un déclenchement doit être idempotent pour une même fenêtre de règlement.
9. Les changements de fréquence ou de seuil ne modifient pas rétroactivement les lots existants.
10. Les modifications administratives sont tracées et peuvent nécessiter une approbation à quatre yeux.

## Exécution attendue

Le planificateur doit :

1. sélectionner les configurations `ACTIVE` arrivées à échéance ;
2. calculer la fenêtre dans le fuseau du commerçant ;
3. vérifier qu’aucun lot n’existe déjà pour cette fenêtre ;
4. calculer le montant éligible ;
5. ignorer la fenêtre si le seuil minimal n’est pas atteint ;
6. créer un lot `DRAFT` via le contrat de règlement ;
7. enregistrer la corrélation entre planification, fenêtre et lot ;
8. publier un événement d’audit et les métriques associées.

## Alignement avec le code

Le contrat de référence est `packages/contracts/src/settlement-schedule.ts` dans `mansa-platform`.

Il expose :

- `SETTLEMENT_FREQUENCIES` ;
- `SETTLEMENT_SCHEDULE_STATUSES` ;
- `SettlementSchedule` et les commandes de création et mise à jour ;
- `createSettlementSchedule` ;
- `updateSettlementSchedule` ;
- les gardes de type associées ;
- le sous-chemin public `@mansa/contracts/settlement-schedule`.

Les tests de référence se trouvent dans `packages/contracts/test/settlement-schedule.test.mjs`.

## Critères d’acceptation

- Une configuration valide est créée dans l’état `ACTIVE`.
- Les fréquences et états inconnus sont rejetés par les gardes de type.
- Les paramètres hebdomadaires et mensuels sont strictement contrôlés.
- Les valeurs hors limites sont rejetées.
- Une configuration peut être suspendue sans être supprimée.
- Le seuil minimal reste un entier sûr non négatif.
- Le contrat est compilable, exporté et couvert par des tests automatisés.
- Aucun secret ni coordonnée bancaire réelle n’est stocké dans la planification.
