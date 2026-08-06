# Promotions et campagnes commerçant

## Objectif

Le module permet à un commerçant de créer des avantages temporaires, mesurables et contrôlables sans modifier directement les prix de référence. Il couvre les remises, cashback et bonus de points, avec ou sans code promotionnel.

## Types de promotion

- remise en pourcentage ;
- remise d’un montant fixe ;
- cashback ;
- bonus de points de fidélité.

Les pourcentages sont exprimés en points de base entiers afin d’éviter les erreurs d’arrondi. Les montants sont toujours représentés en unités mineures entières avec leur devise.

## Cycle de vie

Une promotion suit les états `DRAFT`, `SCHEDULED`, `ACTIVE`, `PAUSED`, `ENDED` ou `CANCELLED`.

- Une promotion en brouillon n’est jamais visible par les clients.
- Une promotion planifiée devient éligible uniquement à partir de sa date de début.
- Une promotion active peut être suspendue immédiatement par un commerçant autorisé ou par l’administration.
- Une promotion terminée ou annulée ne peut plus produire de nouvel avantage.
- Les utilisations déjà accordées restent historisées et ne sont jamais supprimées.

## Déclenchement

Une promotion peut être appliquée automatiquement ou à l’aide d’un code.

Pour un code promotionnel :

- le code est normalisé en majuscules et sans espaces périphériques ;
- l’unicité est garantie dans le périmètre du commerçant et de la période active ;
- les tentatives abusives sont limitées et journalisées ;
- le code ne doit contenir aucune donnée personnelle.

## Éligibilité

Les critères peuvent combiner :

- tous les clients ;
- un ou plusieurs segments ;
- une liste contrôlée de clients ;
- les nouveaux clients ;
- un panier minimum ;
- des produits, catégories ou points de vente autorisés ;
- des produits exclus ;
- le premier achat uniquement.

L’évaluation doit être déterministe et renvoyer un motif stable lorsqu’une promotion est refusée.

## Limites et budget

Une campagne peut définir :

- un nombre total maximum d’utilisations ;
- une limite par client ;
- un budget maximum ;
- un avantage maximum par transaction ;
- une date de début et une date de fin.

Le contrôle des quotas et du budget est atomique. Deux paiements concurrents ne doivent jamais dépasser une limite globale.

## Calcul de l’avantage

1. Vérifier l’état et la fenêtre temporelle.
2. Vérifier le commerçant, le point de vente et les produits concernés.
3. Vérifier le client, son segment et son historique d’utilisation.
4. Vérifier le panier minimum et les exclusions.
5. Calculer l’avantage dans la devise du panier.
6. Appliquer le plafond prévu.
7. Réserver le quota ou le budget de manière atomique.
8. Enregistrer l’utilisation avec la référence du paiement.

Une remise ne peut jamais rendre le montant payé négatif. Les arrondis suivent la règle monétaire configurée pour la devise et doivent être reproductibles.

## Annulation et remboursement

Lorsqu’un paiement est annulé ou remboursé :

- l’utilisation est reliée à l’opération d’origine ;
- le cashback ou les points peuvent être compensés ;
- la remise déjà consommée n’est pas supprimée de l’historique ;
- la restitution d’un quota dépend de la politique configurée ;
- toute correction administrative exige une justification et un audit.

## Sécurité et gouvernance

- Seuls les rôles autorisés peuvent créer, activer, suspendre ou annuler une campagne.
- Les campagnes à fort budget ou très forte remise peuvent nécessiter une double validation.
- Les changements de ciblage, dates, budget et avantage sont audités.
- L’administration peut bloquer une campagne, un commerçant, un pays ou un canal.
- Aucun secret partenaire ni identifiant de paiement complet ne doit apparaître dans les journaux.

## Contrat technique

Le contrat partagé est défini dans `mansa-platform/packages/contracts/src/promotion.ts` et exposé par le sous-chemin `@mansa/contracts/promotion`.

Il couvre :

- les états, types, audiences et modes de déclenchement ;
- la définition de l’avantage ;
- les règles d’éligibilité ;
- les quotas et budgets ;
- les commandes de création, mise à jour et évaluation ;
- le résultat d’évaluation et l’enregistrement d’une utilisation.

## Critères d’acceptation

1. Une promotion inactive ou hors période n’est jamais appliquée.
2. Une remise en pourcentage supérieure à 100 % est rejetée.
3. Une promotion à code exige une comparaison sur le code normalisé.
4. Les devises du panier, du seuil, du plafond et de l’avantage sont cohérentes.
5. Les quotas par client et globaux résistent aux traitements concurrents.
6. Une même utilisation ne peut être enregistrée deux fois pour le même paiement.
7. Toute décision expose un motif exploitable sans révéler de donnée sensible.
8. Les annulations et remboursements produisent une compensation traçable.
9. Les changements administratifs sensibles sont audités.
10. Les applications Client, Commerçant, TPE et Admin utilisent les mêmes règles métier.
