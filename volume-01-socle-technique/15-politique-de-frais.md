# Politique de frais

## Objectif

Une politique de frais décrit le calcul déterministe d’un montant prélevé sur une opération. Elle combine un montant fixe, un taux variable et, si nécessaire, une borne minimale et une borne maximale.

La primitive canonique est `FeePolicy`, exposée par `@mansa/domain` depuis `packages/domain/src/fee-policy.ts`.

## Structure

Une politique contient :

- `fixed` : montant fixe en `Money` ;
- `variable` : taux en `Rate` ;
- `minimum` : montant minimal facultatif ;
- `maximum` : montant maximal facultatif.

Tous les montants d’une même politique utilisent obligatoirement la même devise.

## Calcul canonique

Pour un montant de référence `amount`, le calcul s’effectue dans cet ordre :

1. calculer `variable.applyTo(amount)` ;
2. ajouter le montant fixe ;
3. appliquer le minimum, lorsqu’il existe ;
4. appliquer le maximum, lorsqu’il existe.

Formule :

```text
frais_bruts = fixed + arrondi(amount × variable)
frais = min(max(frais_bruts, minimum), maximum)
```

Les bornes absentes sont ignorées. Le calcul est réalisé uniquement avec des entiers en unités mineures.

## Exemple

Pour une opération de 10 000 XOF, avec 100 XOF fixes et 2,5 % variables :

```text
variable = 250 XOF
frais = 100 XOF + 250 XOF = 350 XOF
```

Configuration JSON :

```json
{
  "fixed": { "minor": "100", "currency": "XOF" },
  "variable": { "basisPoints": "250" },
  "minimum": { "minor": "150", "currency": "XOF" },
  "maximum": { "minor": "1000", "currency": "XOF" }
}
```

## Invariants

- Le montant fixe ne peut pas être négatif.
- Les bornes ne peuvent pas être négatives.
- Le minimum ne peut pas dépasser le maximum.
- Le montant de référence ne peut pas être négatif.
- La devise du montant de référence doit correspondre à celle de la politique.
- Une politique est immuable après sa création.
- Une transaction comptabilisée conserve l’identifiant et la version exacts de la politique appliquée.

## Versionnement métier

La configuration persistée autour de `FeePolicy` doit aussi contenir :

- un identifiant stable ;
- un numéro de version ;
- le produit et le canal concernés ;
- le pays et la devise ;
- le payeur et le bénéficiaire des frais ;
- les dates d’effet ;
- le statut brouillon, approuvé, actif ou archivé ;
- l’auteur, l’approbateur et les références d’audit.

Une modification crée une nouvelle version. Elle ne modifie jamais rétroactivement les frais d’une opération existante.

## Responsabilités

`FeePolicy` réalise uniquement le calcul monétaire. La sélection de la politique applicable selon le produit, le pays, le canal, le partenaire, le segment client ou la date appartient au service de tarification.

Le backend renvoie toujours le détail faisant autorité : montant de référence, part fixe, part variable, taux, minimum, maximum, total final, devise, identifiant et version de règle.

## Critères d’acceptation

- Une politique fixe plus variable produit un résultat exact.
- Le minimum et le maximum sont appliqués après le calcul brut.
- Les incohérences de devise sont rejetées.
- Les montants ou bornes négatifs sont rejetés.
- Un minimum supérieur au maximum est rejeté.
- La sérialisation JSON conserve tous les paramètres sans perte de précision.
- Le code d’interface ne recalcule pas les frais reçus du backend.
