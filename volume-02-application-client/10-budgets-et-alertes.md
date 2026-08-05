# Budgets de dépenses et alertes

## Objectif

Le module de budget permet au client de définir des plafonds de suivi sur ses dépenses sans bloquer automatiquement les paiements. Il sert d’outil de pilotage, d’alerte et de prévention. Les limites transactionnelles exécutoires restent gérées séparément par le moteur de limites et de risque.

## Portée d’un budget

Un budget peut suivre :

- toutes les dépenses d’un portefeuille avec la portée `TOTAL` ;
- une catégorie de dépenses avec la portée `CATEGORY` ;
- un commerçant précis avec la portée `MERCHANT`.

Chaque budget possède un propriétaire, un portefeuille, un nom, un montant limite, un montant consommé, une période, un seuil d’alerte et un statut.

## Périodes

- `WEEKLY` : période hebdomadaire calculée selon le fuseau horaire du client ;
- `MONTHLY` : période mensuelle civile ;
- `CUSTOM` : période définie par une date de début et une date de fin.

Les dates sont stockées de façon non ambiguë et les calculs utilisent le fuseau configuré pour l’affichage et le renouvellement. Un changement de fuseau ne doit pas modifier rétroactivement les consommations comptabilisées.

## Statuts

- `ACTIVE` : consommation calculée et alertes autorisées ;
- `PAUSED` : suivi suspendu sans suppression de l’historique ;
- `ARCHIVED` : budget clôturé et conservé pour consultation.

## Calcul de consommation

1. Une consommation est créée uniquement à partir d’une transaction comptabilisée ou d’un événement financier autorisé.
2. Une transaction annulée, remboursée ou corrigée produit un ajustement explicite ; elle n’est pas supprimée silencieusement.
3. Le montant consommé est reconstruit à partir des consommations liées au budget et ne constitue pas une source de vérité comptable indépendante.
4. La devise du budget doit correspondre à celle du portefeuille et des transactions suivies.
5. Une transaction ne doit être comptée qu’une fois dans un même budget.
6. Le moteur peut rattacher une même transaction à plusieurs budgets distincts, par exemple un budget total et un budget de catégorie.

## Seuils et alertes

Le seuil d’alerte est un pourcentage entier compris entre 1 et 100. À chaque nouvelle consommation, le service calcule :

- le montant projeté après la transaction ;
- le montant restant ;
- l’atteinte du seuil d’avertissement ;
- le dépassement éventuel du budget.

Les notifications doivent être idempotentes afin d’éviter plusieurs alertes identiques pour le même franchissement. Les canaux et préférences du client sont respectés.

## Parcours client

### Création

Le client choisit le portefeuille, le type de budget, le montant, la période et le seuil d’alerte. Pour une catégorie ou un commerçant, l’application vérifie que la cible est valide avant confirmation.

### Consultation

L’écran affiche le montant dépensé, le reste disponible, le pourcentage consommé, la période en cours et les transactions associées. Une progression négative ou incohérente ne doit jamais être masquée : elle déclenche une alerte technique.

### Modification

Le client peut modifier le nom, le montant limite, le seuil d’alerte et le statut. Toute modification est auditée. Une baisse du plafond sous le montant déjà consommé est autorisée uniquement avec un message clair indiquant que le budget est immédiatement dépassé.

### Archivage

L’archivage retire le budget des vues actives sans supprimer son historique. Un budget archivé n’est pas réactivé ; un nouveau budget doit être créé.

## API alignée avec le dépôt plateforme

Contrats maintenus dans :

- `packages/contracts/src/budget.ts` ;
- `packages/contracts/src/budget-api.ts`.

Exports publics :

- `@mansa/contracts/budget` ;
- `@mansa/contracts/budget-api`.

Routes principales :

- `GET /v1/budgets` ;
- `POST /v1/budgets` ;
- `GET /v1/budgets/:budgetId` ;
- `PATCH /v1/budgets/:budgetId` ;
- `GET /v1/budgets/:budgetId/consumptions`.

Les listes sont paginées et filtrables par propriétaire, portefeuille, statut, période et portée.

## Sécurité et confidentialité

- Un client ne peut consulter et modifier que ses propres budgets.
- Les administrateurs doivent disposer d’une permission explicite et d’un motif audité.
- Les libellés de budget ne doivent contenir aucune donnée de carte, code secret ou document d’identité.
- Les alertes ne révèlent pas de détail financier sensible sur un écran verrouillé.
- Les événements restent corrélables avec la transaction sans dupliquer les données sensibles.

## Critères de recette

1. Un budget refuse un montant nul, négatif ou dans une devise incompatible.
2. Un seuil inférieur à 1 ou supérieur à 100 est rejeté.
3. Une portée `CATEGORY` exige une catégorie et une portée `MERCHANT` exige un commerçant.
4. Une transaction comptabilisée n’est consommée qu’une fois par budget.
5. Un remboursement produit un ajustement traçable du montant consommé.
6. Le franchissement d’un seuil n’envoie qu’une alerte idempotente.
7. Un budget en pause n’émet pas de nouvelle alerte.
8. Un budget archivé conserve son historique et ne peut pas être réactivé.
9. Les périodes hebdomadaires, mensuelles et personnalisées sont calculées sans chevauchement.
10. Toute mutation et toute consultation administrative sensible produisent un événement d’audit.
