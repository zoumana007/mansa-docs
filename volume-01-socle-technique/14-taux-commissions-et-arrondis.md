# Taux, commissions et règles d’arrondi

## Objectif

Les commissions, remises, taxes et répartitions proportionnelles doivent produire exactement le même résultat dans le backend, les applications et les traitements comptables. Mansa interdit donc les taux en virgule flottante dans le domaine financier.

La primitive canonique est `Rate`, exposée par `@mansa/domain` depuis `packages/domain/src/rate.ts`.

## Représentation

Un taux est stocké en points de base entiers :

- `1` point de base = 0,01 % ;
- `100` points de base = 1 % ;
- `250` points de base = 2,5 % ;
- `10 000` points de base = 100 %.

Le contrat JSON utilise une chaîne décimale afin de conserver la même convention que les autres valeurs entières sensibles :

```json
{ "basisPoints": "250" }
```

## Domaine autorisé

La primitive `Rate` représente une proportion comprise entre 0 % et 100 % inclus. Les valeurs négatives et supérieures à 10 000 points de base sont rejetées.

Les coefficients qui peuvent dépasser 100 %, les taux de change et les intérêts composés utilisent des objets métier distincts. Ils ne doivent pas détourner `Rate`.

## Calcul d’une commission

Pour un montant `minor` et un taux `basisPoints`, le calcul canonique est :

```text
minor × basisPoints ÷ 10 000
```

Le produit intermédiaire est calculé avec un entier de précision arbitraire. Aucun `number`, `float` ou `double` ne doit intervenir.

## Règle d’arrondi

Mansa applique l’arrondi au plus proche, avec les demi-unités arrondies à l’écart de zéro.

Exemples :

- 0,49 unité mineure devient 0 ;
- 0,50 unité mineure devient 1 ;
- -0,50 unité mineure devient -1.

Cette règle est appliquée par `Rate.applyTo(Money)` et doit être reprise par les adaptateurs qui calculent ou vérifient un montant externe.

## Ordre des opérations

1. Calculer chaque composante réglementaire ou contractuelle à partir du montant de référence documenté.
2. Arrondir chaque composante uniquement au point défini par la règle métier.
3. Enregistrer séparément le brut, chaque commission, chaque taxe et le net.
4. Vérifier l’égalité comptable avant validation de la transaction.

Une interface ne doit jamais recalculer silencieusement une commission affichée. Le backend renvoie le détail faisant autorité.

## Configuration

Toute commission configurable doit préciser :

- identifiant et version de règle ;
- produit, canal, pays et devise concernés ;
- taux en points de base ;
- éventuel minimum et maximum en `Money` ;
- payeur de la commission ;
- bénéficiaire ;
- date de début et de fin ;
- priorité en cas de chevauchement ;
- auteur, approbateur et journal d’audit.

Une modification ne doit jamais altérer rétroactivement une transaction déjà comptabilisée.

## API de domaine

`Rate` fournit :

- `Rate.zero()` ;
- `Rate.ofBasisPoints(value)` ;
- `Rate.fromJSON(value)` ;
- `applyTo(amount)` ;
- `complement()` ;
- `equals(other)` ;
- `toJSON()`.

La primitive est immuable.

## Critères d’acceptation

- 2,5 % de 10 000 XOF produit exactement 250 XOF.
- Les bornes 0 et 10 000 points de base sont acceptées.
- Les valeurs négatives ou supérieures à 10 000 sont rejetées.
- Les demi-unités positives et négatives suivent la même règle symétrique.
- La sérialisation puis désérialisation conserve exactement le taux.
- Le code applicatif n’utilise pas de nombre flottant pour les commissions.
- Les reçus et écritures exposent le taux, la base de calcul et le montant arrondi.
