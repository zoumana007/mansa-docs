# Primitives d’interaction et accessibilité UI

## Objectif

Après les tokens visuels cross-platform, Mansa définit un contrat partagé pour les états d’interaction des contrôles afin d’éviter que chaque application invente ses propres règles pour les boutons, actions sensibles, états désactivés et états de chargement.

Le package exécutable de référence est `@mansa/ui` dans `mansa-platform`.

## Principes

Les primitives restent indépendantes de React, React Native, Next.js ou d’un kit natif. Elles décrivent le comportement attendu ; chaque surface peut ensuite traduire ce contrat vers ses composants réels.

Un contrôle interactif doit toujours fournir un nom accessible non vide. Les espaces de début et de fin sont normalisés avant usage.

Les états partagés sont :

- `idle` ;
- `pressed` ;
- `disabled` ;
- `loading`.

Les états `disabled` et `loading` sont considérés non interactifs. Une application ne doit donc pas laisser passer une action métier pendant ces états uniquement parce que le composant visuel reste monté.

## Intentions et emphases

Intentions communes :

- `neutral` ;
- `primary` ;
- `success` ;
- `warning` ;
- `danger`.

Emphases communes :

- `subtle` ;
- `solid` ;
- `outline`.

Ces valeurs sont sémantiques. Elles ne doivent pas être utilisées pour imposer une couleur hexadécimale directement dans la logique métier.

## Actions destructives

Une action déclarée destructive :

- doit utiliser l’intention `danger` ;
- exige une confirmation au niveau du parcours ;
- ne doit pas être rendue comme une action primaire ordinaire ;
- reste soumise aux règles métier, RBAC, step-up et audit du domaine concerné.

Le contrat UI ne remplace jamais les contrôles serveur.

## Cibles tactiles et focus

Le socle conserve une cible tactile minimale de 44 unités logiques et expose un anneau de focus sémantique avec largeur et décalage partagés. Les implémentations Web doivent conserver un focus visible au clavier ; les applications mobiles doivent préserver une zone d’interaction accessible même lorsque l’élément visuel est plus petit.

## Opacité d’état

Le package expose des valeurs d’opacité communes pour `idle`, `pressed`, `disabled` et `loading`. Ces valeurs servent à harmoniser le rendu, mais l’opacité seule ne doit jamais être le seul signal d’état : texte, icône, attribut accessible ou progression doivent être ajoutés selon le contexte.

## Tests obligatoires

Le package doit tester au minimum :

- la présence des constantes d’accessibilité ;
- la normalisation du nom accessible ;
- le caractère non interactif de `disabled` et `loading` ;
- l’intention `danger` obligatoire pour une action destructive ;
- l’exigence de confirmation pour une action destructive ;
- le refus d’un contrôle sans nom accessible.

## Suite logique

Les prochains composants cross-platform doivent consommer ces primitives au lieu de recopier leurs propres états. Les premiers candidats sont Button/ActionButton, IconButton, champ de formulaire, ligne de liste interactive et actions critiques du portail Admin.
