# UI — formulaires accessibles cross-platform

## Objectif

Les applications Mansa doivent partager un contrat de formulaire cohérent avant le rendu React, React Native, Next.js ou natif. Le package `@mansa/ui` porte la sémantique minimale et indépendante du framework ; chaque application adapte ensuite ce contrat vers les propriétés d’accessibilité de sa plateforme.

## Contrat de champ

Un champ doit disposer au minimum de :

- un `id` stable et non vide ;
- un `label` explicite et non vide ;
- un état `required` lorsque la donnée est obligatoire ;
- un état `disabled` ou `readOnly` lorsque nécessaire ;
- une description facultative ;
- un message d’erreur facultatif.

L’identifiant et le label sont normalisés afin d’éviter les valeurs composées uniquement d’espaces.

## Description et erreurs

Lorsqu’une description existe, sa référence logique est :

```text
<field-id>-description
```

Lorsqu’une erreur existe, sa référence logique est :

```text
<field-id>-error
```

Si les deux existent, le champ doit référencer les deux dans cet ordre. Sur le Web, l’adaptateur doit les traduire vers `aria-describedby`. Sur mobile, l’adaptateur doit produire les propriétés d’accessibilité équivalentes.

Un message d’erreur fait automatiquement passer le statut sémantique du champ à `error`. La couleur seule ne doit jamais être le seul indicateur d’erreur.

## Disabled et read-only

Les deux états ne sont pas interchangeables :

- `disabled` signifie que le champ n’est pas disponible à l’interaction et n’est pas modifiable ;
- `readOnly` signifie que la valeur reste consultable mais n’est pas modifiable.

Le contrat partagé refuse un champ déclaré simultanément `disabled` et `readOnly`, car cette combinaison crée une sémantique ambiguë entre produits.

Dans les deux cas, le champ n’est pas considéré comme interactif par le contrat commun.

## Validation

La validation métier reste côté domaine et serveur. Le package UI ne doit pas décider si un numéro, un montant, un KYC ou un identifiant financier est valide.

Il fournit uniquement la représentation sémantique de l’état du champ. Les règles métier doivent produire un message d’erreur explicite, localisable et compréhensible par l’utilisateur.

## Sécurité

Les interfaces ne doivent jamais :

- exposer un secret dans une description ou une erreur ;
- afficher un PAN complet, CVV, PIN ou secret d’authentification ;
- considérer une validation UI comme une validation serveur ;
- masquer un rejet serveur en transformant seulement l’état visuel du champ.

## Implémentation de référence

Le dépôt `mansa-platform` implémente le contrat dans :

```text
packages/ui/src/form.ts
```

La fonction publique est :

```text
createFieldSemantics()
```

Elle retourne un objet immuable contenant notamment `id`, `label`, `required`, `disabled`, `readOnly`, `interactive`, `status` et `describedBy`.

## Tests obligatoires

La couverture minimale doit vérifier :

- normalisation des labels et identifiants ;
- génération de `describedBy` ;
- statut erreur automatique ;
- comportement `disabled` ;
- comportement `readOnly` ;
- rejet de la combinaison `disabled + readOnly` ;
- rejet des labels et identifiants vides.

## Évolution

Les prochains composants de formulaire peuvent s’appuyer sur ce contrat pour les champs texte, montant, téléphone, recherche, sélection, date et données d’identité. Les composants de framework ne doivent pas dupliquer les règles sémantiques déjà définies dans `@mansa/ui`.
