# Cartes physiques et virtuelles

## 1. Objet

Ce document définit le périmètre fonctionnel et les règles de sécurité du module Cartes de l’application Client Mansa. Il couvre les cartes physiques, virtuelles, temporaires et jetables, sans présumer du partenaire d’émission ou du réseau effectivement retenu en production.

## 2. Types de cartes

- `PHYSICAL` : carte plastique liée à un portefeuille, livrée ou remise selon le parcours du partenaire émetteur.
- `VIRTUAL` : carte numérique durable affichée dans l’application après authentification renforcée.
- `VIRTUAL_TEMPORARY` : carte virtuelle valable pendant une durée limitée ou jusqu’à une date définie.
- `VIRTUAL_DISPOSABLE` : carte virtuelle renouvelée après une utilisation éligible selon les capacités du partenaire.

Les réseaux prévus par le contrat sont `VISA`, `MASTERCARD` et `LOCAL`. L’activation réelle d’un réseau dépend d’un contrat avec une banque, un émetteur et un processeur agréés.

## 3. États du cycle de vie

- `PENDING` : demande créée, émission ou activation en attente.
- `ACTIVE` : carte utilisable selon ses contrôles et limites.
- `FROZEN` : suspension réversible déclenchée par le client ou un acteur autorisé.
- `BLOCKED` : blocage de sécurité ou de conformité nécessitant une procédure contrôlée.
- `EXPIRED` : carte arrivée à expiration, état final.
- `CANCELLED` : carte résiliée, état final.

Une carte expirée ou annulée ne peut pas redevenir active. Tout changement d’état doit être idempotent, audité et associé à un motif lorsqu’il est initié par un administrateur ou un moteur de risque.

## 4. Contrôles d’usage

Le client peut gérer, selon les règles du produit :

- paiements en magasin ;
- paiements en ligne ;
- paiement sans contact ;
- retraits d’espèces ;
- paiements internationaux.

Une désactivation doit prendre effet immédiatement dans Mansa, puis être transmise au processeur. En cas d’indisponibilité du partenaire, l’interface affiche un état de synchronisation sans prétendre que le blocage externe est déjà confirmé.

## 5. Limites de dépense

Les limites configurables sont :

- paiement par transaction ;
- paiement quotidien ;
- retrait par transaction ;
- retrait quotidien ;
- paiement en ligne quotidien.

Tous les montants sont exprimés en unités mineures et dans la devise du portefeuille. Les limites définies par le client ne peuvent jamais dépasser les plafonds du produit, du niveau KYC, du pays, du partenaire ou du moteur de risque.

## 6. Parcours principaux

### Création

1. Le client choisit un portefeuille et un produit disponible.
2. L’application affiche frais, conditions, réseau prévu et délais.
3. Une authentification renforcée est exigée.
4. La commande est envoyée avec une clé d’idempotence.
5. La carte est créée en `PENDING`, puis passe à l’état fourni par le partenaire.

### Gel et dégel

Le gel client est immédiatement demandé et doit être réversible uniquement lorsque le risque et le partenaire l’autorisent. Un blocage de conformité ne peut pas être levé par le client.

### Modification des contrôles et limites

Chaque modification affiche l’ancienne et la nouvelle valeur, exige une confirmation et produit un événement d’audit. Les opérations sensibles peuvent exiger un second facteur.

## 7. Données affichables

L’application peut afficher : nom de la carte, type, réseau, statut, quatre derniers chiffres, expiration, contrôles et limites. Le PAN complet, le cryptogramme et toute donnée équivalente ne doivent jamais être journalisés, envoyés dans les outils analytics ou conservés dans les notifications.

L’affichage de données sensibles doit être temporaire, protégé par authentification renforcée et fourni par un composant certifié du partenaire lorsque requis.

## 8. Contrat API

Le contrat technique est défini dans :

- `packages/contracts/src/card.ts` pour le modèle métier ;
- `packages/contracts/src/card-api.ts` pour les routes et requêtes ;
- `packages/contracts/src/index.ts` pour l’export public du package.

Routes prévues :

- `POST /v1/cards` ;
- `GET /v1/cards` ;
- `GET /v1/cards/:cardId` ;
- `PATCH /v1/cards/:cardId/status` ;
- `PATCH /v1/cards/:cardId/controls` ;
- `PATCH /v1/cards/:cardId/limits`.

Les mutations utilisent une clé d’idempotence et vérifient que la carte appartient au client ou au périmètre autorisé.

## 9. Sécurité et conformité

- Aucun secret de carte ne doit être stocké dans le dépôt ou les journaux.
- Les données de carte sont tokenisées par le partenaire adapté.
- Toute consultation sensible génère un événement de sécurité.
- Les appareils compromis, sessions à risque ou changements récents d’identité peuvent bloquer l’affichage ou la modification.
- Les opérations administratives suivent le RBAC/ABAC, la séparation des responsabilités et la double validation lorsque nécessaire.
- Les obligations PCI DSS et réglementaires sont à valider avec les partenaires avant la production.

## 10. Critères d’acceptation

- Le client ne voit que ses propres cartes.
- La création répétée avec la même clé d’idempotence ne crée pas de doublon.
- Une carte `EXPIRED` ou `CANCELLED` ne peut pas être réactivée.
- Les contrôles et limites sont validés côté serveur, jamais uniquement dans l’application.
- Une limite client supérieure au plafond produit est refusée avec un code d’erreur explicite.
- Les quatre derniers chiffres peuvent être affichés, mais aucune donnée complète ne figure dans les logs.
- Toute mutation produit un audit corrélé.
- Une indisponibilité partenaire est représentée honnêtement par un état en attente ou une erreur réessayable.
- Les routes, méthodes et types documentés correspondent au contrat TypeScript exporté.
