# Budgets, coffres et objectifs d’épargne

## 1. Objectif

Mansa Client doit aider l’utilisateur à organiser son argent sans modifier la vérité comptable du grand livre. Les budgets servent au suivi et aux alertes. Les coffres et objectifs représentent des sommes réellement réservées dans un compte ou sous-compte prévu à cet effet.

Le dépôt `mansa-platform` expose le contrat de domaine `SavingsGoal` pour partager les règles essentielles entre le backend, l’application Client et l’administration.

## 2. Distinction des concepts

### Budget

Un budget est une limite de suivi appliquée à une période et éventuellement à une catégorie. Il n’empêche pas nécessairement une opération. Il peut produire une alerte ou demander une confirmation renforcée.

### Coffre

Un coffre isole une somme du solde immédiatement dépensable. Tout mouvement vers ou depuis un coffre est une transaction comptable traçable.

### Objectif

Un objectif associe un coffre à un nom, une cible monétaire, une devise et éventuellement une date cible. Il affiche la progression et peut recevoir des versements manuels ou automatiques.

## 3. Création d’un objectif

L’utilisateur renseigne :

- un nom ;
- une icône ou illustration non sensible ;
- une devise ;
- un montant cible en unités mineures ;
- une date cible facultative ;
- un versement initial facultatif ;
- une règle d’alimentation facultative.

Le montant cible doit être strictement positif. La date cible doit être postérieure à la création. Un objectif est créé dans l’état `active`, sauf lorsqu’un versement initial atteint exactement la cible.

## 4. États

Les états partagés sont :

- `active` : contributions autorisées ;
- `paused` : automatisations suspendues, retraits encore possibles selon la politique ;
- `completed` : cible atteinte exactement ;
- `cancelled` : objectif arrêté définitivement.

Une transition produit un événement de domaine et une trace d’audit lorsqu’elle entraîne un mouvement financier ou une action administrative.

## 5. Contributions

Une contribution doit :

- être strictement positive ;
- utiliser la même devise que l’objectif ;
- ne pas dépasser la cible ;
- référencer la transaction comptable associée ;
- être idempotente ;
- respecter les limites transactionnelles ;
- être refusée lorsque l’objectif n’est pas actif.

Lorsque le total atteint exactement la cible, l’objectif passe automatiquement à `completed`.

## 6. Retraits

Un retrait depuis un coffre :

- ne peut pas dépasser le montant disponible ;
- doit être positif ;
- est interdit depuis un objectif terminé ou annulé sauf procédure explicite de clôture ;
- doit passer par le grand livre ;
- doit respecter les contrôles de risque, de conformité et d’authentification ;
- met à jour la progression uniquement après confirmation comptable.

Le retrait ne doit jamais être simulé par une simple modification du champ de progression.

## 7. Règles d’alimentation automatique

Les règles possibles comprennent :

- montant fixe quotidien, hebdomadaire ou mensuel ;
- pourcentage d’un encaissement ;
- arrondi d’un paiement ;
- répartition automatique d’un salaire ;
- versement à une date choisie ;
- transfert lorsque le solde principal dépasse un seuil.

Chaque règle possède un état, une prochaine date d’exécution, une limite, une source de fonds et une politique en cas de solde insuffisant. Une règle ne doit pas créer de découvert involontaire.

## 8. Budgets de dépenses

Un budget peut être défini par :

- catégorie ;
- commerçant ;
- carte ;
- portefeuille ;
- période ;
- pays ;
- projet ;
- membre d’une équipe.

L’application affiche le montant prévu, le montant consommé, les opérations en attente, le reste disponible et la date de réinitialisation.

Les autorisations en attente sont affichées séparément afin d’éviter de présenter une estimation comme un solde comptable définitif.

## 9. Alertes

Les seuils configurables peuvent être 50 %, 80 %, 100 % ou toute valeur autorisée par l’administration. Les alertes utilisent les préférences de communication du client et ne doivent pas révéler de montant sensible sur un écran verrouillé sans consentement.

Les événements minimums sont :

- seuil de budget atteint ;
- contribution réussie ou rejetée ;
- objectif terminé ;
- automatisation suspendue ;
- retrait effectué ;
- date cible proche ;
- échec répété d’une règle automatique.

## 10. Partage et objectifs collectifs

Une version ultérieure peut permettre un objectif partagé. Elle doit définir :

- propriétaire légal des fonds ;
- membres et permissions ;
- visibilité des contributions ;
- conditions de retrait ;
- gestion du départ d’un membre ;
- litiges ;
- KYC et limites de chaque participant ;
- règles fiscales ou réglementaires applicables.

Aucun objectif collectif ne doit être activé en production avant validation juridique et conformité.

## 11. Administration

L’administration peut configurer :

- devises et montants minimums ou maximums ;
- nombre d’objectifs par client ;
- périodicités ;
- règles d’arrondi ;
- seuils d’alerte ;
- catégories de budget ;
- retraits autorisés ;
- frais éventuels ;
- fonctionnalités par pays et segment ;
- maintenance et arrêt d’urgence.

Les changements sont versionnés et ne modifient pas rétroactivement un contrat client sans règle explicite.

## 12. Sécurité et comptabilité

- Les montants utilisent des entiers en unités mineures.
- Les `bigint` sont sérialisés sous forme de chaînes afin d’éviter toute perte de précision.
- Le grand livre reste la source de vérité des fonds.
- Les agrégats de progression peuvent être recalculés à partir des écritures.
- Toute automatisation utilise une clé d’idempotence.
- Les dates sont copiées pour éviter les mutations externes.
- Aucun secret bancaire ou partenaire n’est conservé dans ce module.

## 13. Critères de recette

- Un objectif invalide ou sans propriétaire est refusé.
- Une devise ne respectant pas le format ISO à trois lettres est refusée.
- Une cible nulle ou négative est refusée.
- Une contribution nulle, négative ou supérieure au restant est refusée.
- La cible atteinte exactement entraîne l’état `completed`.
- Un objectif en pause ne reçoit pas de contribution.
- Un objectif en pause peut être repris.
- Un objectif terminé ou annulé ne peut pas être annulé à nouveau.
- Un retrait supérieur au montant disponible est refusé.
- Les montants sérialisés ne perdent aucune précision.
- La progression affichée est cohérente avec les transactions confirmées.
- La documentation et le contrat `SavingsGoal` utilisent les mêmes états.
