# Volume 1 — Contrats partagés

## 1. Objectif

Le paquet `@mansa/contracts` définit le langage commun entre les applications, l’API, les travailleurs asynchrones et les intégrations. Il contient uniquement des types, constantes, commandes, réponses et fonctions de validation légères. Il ne contient ni accès aux données, ni logique dépendante d’un framework, ni secret.

## 2. Domaines couverts

Le socle de contrats couvre progressivement :

- monnaie et devises ;
- idempotence et références transactionnelles ;
- erreurs API et pagination ;
- audit ;
- identité, sessions et authentification ;
- KYC ;
- portefeuilles et transferts ;
- paiements et demandes d’argent ;
- cartes ;
- commerçants et règlements ;
- terminaux TPE ;
- administration et approbations ;
- notifications ;
- support.

## 3. Règles de conception

1. Les dates traversant une API sont des chaînes ISO 8601 en UTC.
2. Les montants sont des unités mineures entières et sont associés à une devise.
3. Les statuts sont des unions fermées dérivées de tableaux constants.
4. Les commandes sensibles transportent une clé d’idempotence ou une référence de corrélation.
5. Les références externes sont facultatives tant qu’un fournisseur ne les a pas attribuées.
6. Les informations sensibles sont masquées dans les réponses destinées aux interfaces.
7. Une rupture de contrat exige une nouvelle version ou une migration coordonnée.

## 4. Compatibilité

L’ajout d’un champ facultatif ou d’une nouvelle valeur gérée de façon tolérante peut être compatible. La suppression d’un champ, son changement de sens, ou l’ajout d’une valeur non gérée par les consommateurs est une rupture.

Les applications doivent ignorer les champs inconnus et afficher un comportement sûr lorsqu’un statut plus récent n’est pas encore reconnu.

## 5. Notifications

Les notifications sont séparées du contenu final envoyé par un fournisseur. Une commande référence un modèle versionné, des variables et des canaux autorisés. Le résultat conserve un statut par livraison, une destination masquée et, lorsque disponible, une référence fournisseur.

Les préférences marketing ne peuvent pas désactiver les alertes de sécurité ou messages légalement obligatoires. Les canaux soumis au consentement sont filtrés avant mise en file.

## 6. Support

Le contrat support définit les catégories, priorités, statuts, messages et références transactionnelles. Les notes internes ne font pas partie du contrat client. Les pièces jointes sont référencées par identifiant et contrôlées par autorisation avant téléchargement.

## 7. Validation continue

Chaque modification du paquet doit satisfaire :

- compilation TypeScript stricte ;
- absence d’import de framework ;
- vérification des exports publics ;
- tests unitaires des fonctions de validation ;
- revue de compatibilité avec les consommateurs ;
- mise à jour de cette documentation lorsque le domaine change.
