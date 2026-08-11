# UI — progression et chargement accessibles cross-platform

## Objectif

Toutes les applications Mansa doivent représenter les opérations longues, synchronisations, imports, chargements, traitements et confirmations différées avec une sémantique cohérente et accessible. Le comportement doit être partagé entre Web, mobile et interfaces spécialisées sans dépendance à React, React Native ou un framework natif.

## Contrat commun

Le package `@mansa/ui` expose `createProgressSemantics()` avec :

- un identifiant stable ;
- un libellé obligatoire ;
- un état `idle`, `running`, `success` ou `error` ;
- une plage `min` / `max` ;
- une valeur facultative pour une progression déterminée ;
- un mode indéterminé lorsqu’aucune valeur fiable n’existe ;
- un rôle logique `progressbar` ;
- une priorité d’annonce de fin ou d’erreur.

## Progression déterminée

Une progression déterminée ne peut être utilisée que lorsque le produit connaît une valeur réelle et fiable. La valeur doit être finie et comprise dans la plage autorisée. `max` doit toujours être strictement supérieur à `min`.

Exemples adaptés :

- import de 40 fichiers sur 100 ;
- téléchargement dont la taille totale est connue ;
- traitement batch dont le nombre d’éléments est connu.

## Progression indéterminée

Lorsqu’aucun pourcentage fiable n’existe, le système doit utiliser le mode indéterminé et ne jamais fabriquer un faux pourcentage.

Exemples :

- attente d’une réponse partenaire bancaire ;
- vérification KYC distante ;
- synchronisation réseau dont la durée est inconnue ;
- validation serveur d’une opération.

## Accessibilité

Une progression en cours ne doit pas déclencher une annonce vocale à chaque variation de valeur. Le contrat partagé utilise donc une région live désactivée pendant l’exécution normale.

À la fin :

- `success` peut être annoncé poliment ;
- `error` doit être annoncé assertivement ;
- une couleur, un spinner ou une animation seuls ne suffisent jamais à transmettre l’état.

Sur le Web, les adaptateurs doivent traduire la sémantique vers `role=progressbar`, les bornes et valeurs ARIA appropriées. Sur mobile, les API d’accessibilité natives équivalentes doivent être utilisées.

## Contraintes financières et de sécurité

Une interface ne doit jamais passer en `success` uniquement parce qu’une animation locale est terminée. Pour un paiement, transfert, retrait, dépôt, KYC ou autre action critique, l’état final doit venir du résultat métier confirmé par le backend ou la source d’autorité prévue.

Le composant ne doit jamais exposer :

- token ;
- secret ;
- PAN complet ;
- PIN ;
- CVV ;
- identifiant interne sensible ;
- stack trace ;
- détail technique exploitable.

Un identifiant de corrélation non sensible peut être affiché séparément lorsque le support en a besoin.

## Implémentation de référence

Dans `mansa-platform` :

```text
packages/ui/src/progress.ts
packages/ui/test/progress.test.mjs
```

Fonction publique :

```text
createProgressSemantics()
```

## Tests obligatoires

La couverture doit vérifier :

- normalisation de l’identifiant et du libellé ;
- progression déterminée valide ;
- progression indéterminée sans valeur ;
- annonce polie de la réussite ;
- annonce assertive de l’erreur ;
- rejet d’une plage invalide ;
- rejet d’une valeur hors bornes ;
- rejet d’une valeur fournie avec le mode indéterminé ;
- rejet d’un mode déterminé sans valeur ;
- rejet des identifiants et libellés vides.
