# Représentation des montants financiers

## Objectif

Tous les montants Mansa sont représentés en unités monétaires mineures avec des entiers. Les nombres à virgule flottante sont interdits dans le domaine financier afin d’éviter les erreurs d’arrondi et les divergences entre services.

La primitive canonique est exposée par `@mansa/domain` dans `packages/domain/src/money.ts`.

## Modèle

Un montant contient exactement :

- `minor` : entier signé sérialisé en chaîne décimale ;
- `currency` : code devise ISO 4217 en trois lettres majuscules.

Exemples :

```json
{ "minor": "1500", "currency": "XOF" }
```

```json
{ "minor": "1099", "currency": "EUR" }
```

Pour le XOF, qui ne possède pas de sous-unité utilisée dans les paiements courants, `1500` représente 1 500 FCFA. Pour l’euro, `1099` représente 10,99 EUR.

## Règles métier

1. Deux montants ne peuvent être additionnés ou soustraits que s’ils utilisent la même devise.
2. Une différence de devise provoque une erreur explicite ; aucune conversion implicite n’est autorisée.
3. Les conversions de devise passent par un service dédié avec taux, source, horodatage et règle d’arrondi enregistrés.
4. La sérialisation JSON utilise une chaîne pour `minor` afin de préserver les grands entiers dans tous les clients.
5. Les montants peuvent être négatifs dans les écritures comptables, remboursements et corrections, mais les commandes métier décident si cette valeur est autorisée dans leur contexte.
6. Le code devise est validé au format `^[A-Z]{3}$`.

## API de domaine

La primitive `Money` fournit :

- `Money.zero(currency)` ;
- `Money.ofMinor(minor, currency)` ;
- `Money.fromJSON(value)` ;
- `add`, `subtract`, `negate` et `absolute` ;
- `isZero`, `isPositive`, `isNegative` et `equals` ;
- `toJSON`.

Elle est immuable : chaque opération retourne une nouvelle valeur.

## Interdictions

- Ne jamais utiliser `number`, `float`, `double` ou un type SQL flottant pour un montant.
- Ne jamais déduire la devise depuis le pays sans la transmettre explicitement.
- Ne jamais arrondir silencieusement dans un contrôleur ou une interface utilisateur.
- Ne jamais comparer deux montants de devises différentes comme s’ils étaient équivalents.
- Ne jamais stocker un taux de change sans sa date, sa source et sa paire de devises.

## Stockage cible

Dans PostgreSQL, les valeurs monétaires métier utilisent une colonne entière 64 bits lorsque la plage est suffisante. Les agrégats ou cas dépassant cette plage utilisent `NUMERIC(38,0)`. La devise est stockée séparément et contrôlée par les règles de domaine.

Le grand livre en partie double conserve le montant signé, la devise, le compte, la transaction, la date effective et la date d’enregistrement.

## Critères d’acceptation

- La primitive compile en TypeScript strict.
- Une addition ou soustraction de devises différentes échoue.
- La sérialisation puis désérialisation conserve exactement la valeur entière.
- Aucun contrat financier public n’expose un montant flottant.
- Les adaptateurs partenaires convertissent leurs formats vers `Money` à leur frontière.
- Les applications formatent les montants pour l’affichage sans modifier la valeur métier.
