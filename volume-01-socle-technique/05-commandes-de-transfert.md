# Commande de transfert

## Objectif

La commande de transfert représente une intention validée de déplacer un montant entre deux wallets Mansa. Elle ne modifie aucun solde directement : son exécution doit passer par un service applicatif transactionnel, le ledger en partie double et le mécanisme d’idempotence.

## Données obligatoires

- `transferId` : identifiant unique du transfert ;
- `sourceWalletId` : wallet débité ;
- `destinationWalletId` : wallet crédité ;
- `amount` : montant strictement positif en unités mineures avec devise explicite ;
- `idempotencyKey` : clé empêchant l’exécution multiple d’une même demande.

La référence métier est facultative, mais ne peut pas être vide lorsqu’elle est fournie.

## Invariants

1. Le wallet source et le wallet destination sont différents.
2. Le montant est strictement positif.
3. Tous les identifiants sont non vides après normalisation.
4. La devise est portée par l’objet `Money`.
5. La commande est immuable et sérialisable.
6. La commande seule n’autorise aucune mutation financière.

## Exécution cible

Le futur service applicatif de transfert devra, dans une transaction atomique :

1. réserver la clé d’idempotence ;
2. charger et verrouiller les deux wallets ;
3. vérifier leur statut, leur devise et le solde disponible ;
4. calculer les frais applicables ;
5. créer l’écriture équilibrée du ledger ;
6. enregistrer le transfert et son état ;
7. publier les événements via l’outbox ;
8. produire une trace d’audit corrélée.

## État du code

Le paquet `@mansa/domain` expose désormais `TransferCommand` et `InvalidTransferCommandError`. Les tests couvrent la normalisation, la sérialisation, le montant invalide, l’auto-transfert et les champs vides.

## Hypothèse à valider

Les plafonds, frais, règles de change, délais de règlement et contrôles réglementaires varieront selon le pays, le type de compte, le canal et le partenaire. Ils ne doivent pas être figés dans la commande de domaine.
