# Calcul des soldes comptables

## Objectif

Le solde affiché dans Mansa est une projection dérivée des lignes du grand livre. Il ne remplace jamais les écritures comptables immuables et ne doit pas être utilisé comme source de vérité isolée.

La primitive `calculateLedgerBalance` est exposée par `@mansa/domain` depuis `packages/domain/src/ledger-balance.ts`.

## Convention de signe

Les comptes `ASSET` et `EXPENSE` ont un sens normal débiteur :

```text
solde = total des débits - total des crédits
```

Les comptes `LIABILITY`, `EQUITY` et `REVENUE` ont un sens normal créditeur :

```text
solde = total des crédits - total des débits
```

Un solde négatif indique que le compte présente un sens contraire à son sens normal. Cette situation peut être valide dans certains cas, mais elle doit être expliquée par les règles métier et surveillée pour les comptes sensibles.

## Entrées et sortie

Le calcul reçoit :

- l’identifiant du compte ciblé ;
- le type comptable du compte ;
- une collection de lignes du grand livre.

Il retourne une projection immuable contenant :

- `debitMinor` : total débité en unité monétaire mineure ;
- `creditMinor` : total crédité en unité monétaire mineure ;
- `balanceMinor` : solde signé selon le type de compte.

Les lignes d’autres comptes sont ignorées. Les montants flottants sont interdits : le calcul utilise exclusivement des entiers `bigint` en unité mineure.

## Règles d’intégrité

- L’identifiant du compte est obligatoire.
- Une ligne ciblée doit porter un montant strictement positif.
- Le côté est obligatoirement `DEBIT` ou `CREDIT`.
- Le type de compte provient du catalogue du plan comptable.
- La devise n’est pas mélangée dans une projection ; le filtrage mono-devise doit être garanti par le dépôt ou le service applicatif.
- La fonction ne modifie jamais les lignes reçues.

## Persistance et performances

En production, le solde courant peut être matérialisé dans une table de projection pour accélérer la lecture. Cette projection doit être mise à jour dans la même transaction logique que l’écriture ou par un consommateur idempotent capable de reconstruire intégralement l’état depuis le journal.

La reconstruction complète reste obligatoire pour :

- les audits ;
- la correction d’une projection corrompue ;
- la migration d’un modèle de solde ;
- les contrôles de rapprochement ;
- la génération de preuves comptables.

Pour les comptes à fort volume, les lectures doivent être paginées ou agrégées côté base. Le domaine ne doit pas charger indéfiniment toutes les écritures en mémoire.

## Concurrence et cohérence

Deux opérations simultanées ne doivent pas produire un solde incohérent. La couche applicative doit utiliser une stratégie explicite : verrouillage de ligne, version optimiste, séquence de journal ou traitement sérialisé par compte.

Le contrôle de disponibilité d’un wallet doit être effectué dans la même transaction que la réservation ou l’écriture, jamais uniquement à partir d’un solde lu auparavant.

## Critères d’acceptation

- Un actif avec 10 000 unités débitées et 2 500 créditées présente un solde de 7 500.
- Un compte de revenu avec 7 500 unités créditées présente un solde de 7 500.
- Les lignes appartenant à d’autres comptes sont ignorées.
- Un compte sans ligne présente des totaux et un solde nuls.
- Un identifiant vide est rejeté.
- Un montant nul ou négatif sur une ligne ciblée est rejeté.
- Les tests automatisés couvrent les deux sens normaux et les validations essentielles.

## Hypothèse à valider

Le traitement définitif des soldes disponibles, soldes comptables, réservations, autorisations carte et retenues réglementaires sera précisé avec la banque partenaire et les prestataires de paiement. Ces concepts ne doivent pas être confondus dans une seule valeur de solde.
