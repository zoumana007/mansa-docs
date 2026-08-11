# UI — feedback accessible cross-platform

## Objectif

Toutes les applications Mansa doivent annoncer de manière cohérente les informations, confirmations, avertissements et erreurs, y compris pour les utilisateurs de technologies d’assistance. Le contrat est indépendant de React, React Native, Next.js ou d’un SDK natif.

## Contrat commun

Le package `@mansa/ui` expose `createFeedbackSemantics()` avec :

- un identifiant stable ;
- un message obligatoire ;
- un titre facultatif ;
- un ton `info`, `success`, `warning` ou `error` ;
- la persistance ;
- la possibilité de dismissal ;
- un rôle logique ;
- une priorité d’annonce.

## Priorité d’annonce

Les messages `info`, `success` et `warning` utilisent une annonce polie : ils ne doivent pas interrompre inutilement la lecture en cours.

Les erreurs utilisent une annonce assertive et un rôle d’alerte, car une erreur de paiement, d’authentification ou une action impossible peut nécessiter une attention immédiate.

Une couleur ou une icône seule ne suffit jamais à transmettre le statut.

## Persistance

Une erreur est persistante par défaut. Un message persistant ne peut pas être simultanément dismissible dans le contrat partagé : il doit rester visible jusqu’à résolution, remplacement explicite ou navigation contrôlée.

Les messages non persistants peuvent être dismissibles, mais les produits doivent éviter de masquer automatiquement une information nécessaire à une décision financière ou de sécurité.

## Adaptateurs

Sur le Web, les adaptateurs traduisent les propriétés logiques vers `role=status|alert` et les attributs live appropriés. Sur mobile, ils utilisent les API d’accessibilité équivalentes.

Le package partagé ne doit pas contenir de dépendance à un framework d’interface.

## Sécurité et confidentialité

Les messages visibles ne doivent jamais inclure :

- PIN ;
- CVV ;
- PAN complet ;
- token d’authentification ;
- clé API ou secret ;
- stack trace ;
- information interne permettant d’exploiter le système.

Les détails techniques doivent être enregistrés dans la télémétrie serveur avec un identifiant de corrélation. Le message utilisateur doit rester explicite, localisable et non sensible.

## Implémentation

Référence dans `mansa-platform` :

```text
packages/ui/src/feedback.ts
packages/ui/test/feedback.test.mjs
```

Fonction publique :

```text
createFeedbackSemantics()
```

## Tests obligatoires

La couverture doit vérifier :

- normalisation des textes ;
- `status` + `polite` pour information/succès/avertissement ;
- `alert` + `assertive` pour les erreurs ;
- persistance par défaut des erreurs ;
- rejet d’un feedback à la fois persistant et dismissible ;
- rejet des identifiants et messages vides.
