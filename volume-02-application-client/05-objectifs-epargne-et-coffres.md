# Objectifs d’épargne et coffres

## 1. Objet

Ce document définit le module d’épargne de l’application Client Mansa. Il permet à un client de créer des objectifs, d’y transférer de l’argent manuellement ou automatiquement et de suivre sa progression sans confondre l’épargne applicative avec un produit bancaire rémunéré.

## 2. Principes

- Un objectif est rattaché à un portefeuille et utilise la même devise.
- Les montants sont exprimés en unités mineures avec le type partagé `Money`.
- Les fonds affectés à un objectif restent traçables dans le grand livre.
- Aucun rendement, intérêt ou garantie n’est affiché sans produit réglementé et partenaire contractuel.
- Toute mutation financière utilise une clé d’idempotence.

## 3. Cycle de vie

Les états sont :

- `ACTIVE` : objectif ouvert et finançable ;
- `PAUSED` : alimentation automatique suspendue ;
- `COMPLETED` : montant cible atteint ;
- `CANCELLED` : objectif fermé par un acteur autorisé.

Un objectif terminé peut rester visible dans l’historique. Son éventuelle réouverture doit être explicitement autorisée par la politique produit et auditée.

## 4. Création d’un objectif

Le client renseigne :

- un nom ;
- un portefeuille source ;
- un montant cible ;
- éventuellement une date cible.

Le serveur vérifie la devise, les limites produit, le niveau KYC, la longueur du nom et la cohérence de la date. La répétition d’une requête avec la même clé d’idempotence ne crée pas de doublon.

## 5. Alimentation manuelle

L’alimentation transfère une somme du portefeuille source vers la position comptable liée à l’objectif. L’opération doit :

1. vérifier le solde disponible et les limites ;
2. créer une écriture équilibrée dans le grand livre ;
3. mettre à jour la progression de manière atomique ;
4. produire une transaction et un événement d’audit corrélés ;
5. gérer les reprises sans double débit.

## 6. Alimentation automatique

Une règle peut prévoir une alimentation `DAILY`, `WEEKLY` ou `MONTHLY`. Elle contient un montant, une prochaine date d’exécution et un état actif ou inactif.

Chaque exécution possède sa propre clé d’idempotence déterministe. En cas de solde insuffisant, le système ne met jamais le portefeuille à découvert, enregistre l’échec et applique la politique de nouvelle tentative configurée.

## 7. Retrait et annulation

Le retrait renvoie tout ou partie des fonds vers un portefeuille autorisé de même devise. Les règles produit peuvent imposer un délai, une authentification renforcée, un motif ou une restriction temporaire liée au risque.

L’annulation d’un objectif ne doit pas supprimer son historique. Le traitement du solde restant est explicite et atomique : remboursement, maintien jusqu’à validation ou blocage conformité.

## 8. Affichage client

L’application affiche au minimum : nom, montant cible, montant épargné, progression, statut, date cible éventuelle et prochaine alimentation. Les soldes en attente ou bloqués sont distingués du montant réellement disponible.

Aucune promesse de rendement ne doit être déduite d’une animation, d’un texte marketing ou d’une projection non contractuelle.

## 9. Contrat technique

Le dépôt plateforme définit :

- `packages/contracts/src/savings.ts` ;
- `packages/contracts/src/savings-api.ts` ;
- les exports publics dans `packages/contracts/src/index.ts`.

Routes prévues :

- `POST /v1/savings/goals` ;
- `GET /v1/savings/goals` ;
- `GET /v1/savings/goals/:goalId` ;
- `POST /v1/savings/goals/:goalId/fund` ;
- `POST /v1/savings/goals/:goalId/withdraw` ;
- `PUT /v1/savings/goals/:goalId/funding-rule`.

## 10. Sécurité et conformité

- Le client ne consulte et ne modifie que ses objectifs.
- Les opérations administratives utilisent RBAC/ABAC et journal d’audit.
- Les fonds bloqués par une décision conformité ne sont pas retirables.
- Les tâches planifiées sont corrélées, observables et réessayables sans double mouvement.
- Les notifications ne contiennent aucune donnée sensible inutile.
- La qualification juridique du produit doit être validée avant activation en production.

## 11. Critères d’acceptation

- La création répétée avec la même clé ne produit qu’un objectif.
- Une alimentation ne peut pas dépasser le solde disponible ni les plafonds applicables.
- Toutes les écritures financières sont équilibrées et corrélées à la transaction.
- Une exécution automatique répétée ne débite jamais deux fois.
- Un objectif dans une devise ne peut pas être alimenté depuis un portefeuille d’une autre devise sans conversion explicite.
- Le retrait met à jour le grand livre et la progression atomiquement.
- L’historique reste disponible après achèvement ou annulation.
- Les routes, méthodes et types documentés correspondent aux contrats TypeScript.
