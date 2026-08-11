# UI — navigation accessible cross-platform

## Objectif

Toutes les applications Mansa doivent proposer des navigations principales, secondaires, fils d’Ariane et onglets avec une sémantique cohérente, accessible et indépendante du framework. Le contrat doit être réutilisable sur Web, mobile, TPE et interfaces spécialisées sans confondre rendu visuel et autorisation métier.

## Contrat commun

Le package `@mansa/ui` expose `createNavigationSemantics()` avec :

- un identifiant stable ;
- un libellé accessible obligatoire ;
- un type `primary`, `secondary`, `breadcrumb` ou `tabs` ;
- au moins un élément ;
- des identifiants d’éléments uniques ;
- un état courant facultatif et unique ;
- un état désactivé ;
- une indication de focalisation ;
- un rôle logique `navigation` ou `tablist`.

## Élément courant

Une navigation ne doit contenir qu’un seul élément courant. Cet état doit provenir de la route ou de l’état applicatif réel et ne jamais être simulé uniquement par une couleur.

L’élément courant :

- reste focalisable ;
- ne peut pas être désactivé ;
- expose une sémantique équivalente à `aria-current=page` lorsque l’adaptateur le permet.

## Éléments désactivés

Un élément désactivé :

- ne doit pas être focalisable ;
- ne doit pas déclencher d’action ;
- ne doit pas être utilisé comme unique mécanisme d’autorisation ;
- doit conserver un libellé compréhensible lorsqu’il reste visible.

## Onglets

Le type `tabs` expose le rôle logique `tablist`. Les adaptateurs doivent compléter la sémantique native requise pour les onglets, notamment le contrôle du focus et la relation entre onglet et panneau.

Le contrat partagé ne doit pas imposer un framework ou une API de rendu spécifique.

## Accessibilité

La navigation doit :

- conserver un ordre logique conforme à l’ordre de lecture ;
- avoir un nom accessible lorsque plusieurs régions de navigation existent ;
- rendre l’état courant perceptible autrement que par la couleur ;
- éviter les pièges de focus ;
- utiliser les primitives natives de la plateforme lorsque possible ;
- rester utilisable avec clavier, lecteur d’écran et technologies d’assistance.

## Sécurité et autorisation

La visibilité d’un élément de navigation ne constitue jamais une autorisation.

Les applications doivent continuer à appliquer :

- RBAC/ABAC ;
- vérification de session ;
- droits tenant/organisation ;
- contrôles backend ;
- step-up ou MFA pour les actions sensibles lorsque prévu.

Masquer un lien d’administration ne suffit pas à protéger la route correspondante.

La structure de navigation ne doit jamais exposer :

- secret ;
- token ;
- PAN complet ;
- PIN ;
- CVV ;
- identifiant interne sensible ;
- route technique confidentielle non destinée à l’utilisateur.

## Implémentation de référence

Dans `mansa-platform` :

```text
packages/ui/src/navigation.ts
packages/ui/test/navigation.test.mjs
```

Fonction publique :

```text
createNavigationSemantics()
```

## Tests obligatoires

La couverture doit vérifier :

- normalisation des identifiants et libellés ;
- rôle de navigation principal ;
- rôle `tablist` pour les onglets ;
- élément courant unique ;
- élément désactivé non focalisable ;
- rejet de plusieurs éléments courants ;
- rejet d’un élément courant désactivé ;
- rejet des identifiants dupliqués ;
- rejet d’une navigation vide ;
- rejet des champs obligatoires vides.
