# Gouvernance des contrats API

## 1. Finalité

Les contrats API de Mansa doivent permettre aux applications mobiles, interfaces web, terminaux, workers et partenaires d'évoluer sans divergence silencieuse. La source technique partagée se trouve dans `mansa-platform/packages/contracts`.

## 2. Sources de vérité

- La documentation fonctionnelle et les règles métier sont définies dans `mansa-docs`.
- Les types sérialisables, routes, méthodes, commandes et réponses sont définis dans `@mansa/contracts`.
- L'implémentation backend doit satisfaire les deux sources.
- En cas d'écart, aucune mise en production n'est autorisée avant correction ou décision d'architecture documentée.

## 3. Versionnement

Les routes publiques utilisent un préfixe de version, initialement `/v1`.

Une évolution additive peut rester dans la version courante lorsqu'elle ne modifie pas le comportement existant. Une rupture exige une nouvelle version et une période de coexistence définie.

Sont considérés comme ruptures :

- suppression ou renommage d'un champ ;
- changement de type ou d'unité ;
- modification d'une valeur par défaut ;
- changement de sémantique d'un statut ;
- ajout d'une obligation qui rend invalide une requête auparavant valide ;
- modification d'une règle d'autorisation observable par un partenaire.

## 4. Enveloppes de réponse

Toute réponse réussie contient les données métier et les métadonnées nécessaires à la corrélation. Toute erreur publique contient un code stable, un message exploitable, un identifiant de corrélation et, lorsque nécessaire, des détails non sensibles.

Une erreur ne doit jamais exposer :

- une trace d'exécution ;
- une requête SQL ;
- un secret ou jeton ;
- un numéro de carte complet ;
- un document KYC ;
- une information personnelle sans nécessité fonctionnelle.

## 5. Idempotence

Une clé d'idempotence est obligatoire pour toute création ou mutation financière, toute commande administrative sensible et tout traitement partenaire susceptible d'être rejoué.

Le serveur doit :

1. associer la clé à l'acteur, l'opération et la charge utile normalisée ;
2. retourner le résultat initial lors d'un rejeu identique ;
3. refuser la réutilisation avec une charge utile différente ;
4. conserver la preuve assez longtemps pour couvrir les délais de rejeu et de réconciliation ;
5. tracer les conflits sans exposer la charge utile sensible.

## 6. Pagination et filtres

Les listes à fort volume utilisent une pagination par curseur. Les limites sont bornées côté serveur. Les filtres, tris et périodes acceptés sont explicitement définis dans les contrats.

Une page doit indiquer au minimum :

- les éléments retournés ;
- la présence éventuelle d'une page suivante ;
- le curseur suivant lorsqu'il existe ;
- la limite appliquée.

## 7. Dates, devises et montants

- Les dates d'échange sont en ISO 8601 UTC.
- Les devises suivent un code stable validé par configuration pays.
- Les montants sont exprimés en unités mineures entières.
- Aucun flottant n'est accepté pour une valeur financière.
- Une opération multidevise expose séparément montant source, montant destination, taux, frais et horodatage du devis.

## 8. États et transitions

Les statuts publics sont des ensembles fermés documentés. Les transitions invalides sont rejetées avec un code d'erreur stable. Un état final ne peut pas être modifié directement ; une correction financière utilise une opération compensatoire.

Les clients doivent afficher un état inconnu de manière sûre au lieu de considérer automatiquement l'opération comme réussie.

## 9. Sécurité et autorisation

Chaque route documente :

- le type d'acteur autorisé ;
- les permissions requises ;
- le niveau d'authentification ;
- les obligations éventuelles de double validation ;
- la portée pays, organisation, commerce ou compte ;
- les événements d'audit produits.

L'autorisation est vérifiée côté serveur même lorsque l'interface masque une action.

## 10. Webhooks et partenaires

Les webhooks sont signés, horodatés, rejouables et idempotents. Chaque livraison possède un identifiant, un statut, un compteur de tentatives et une politique de réessai bornée.

Les adaptateurs partenaires traduisent les formats externes vers les contrats internes. Un SDK ou format fournisseur ne doit pas se propager dans le domaine.

## 11. Publication des contrats techniques

Un contrat API stable n'est considéré comme réellement disponible pour les consommateurs que lorsque sa publication est cohérente dans le package partagé.

Pour chaque fichier `packages/contracts/src/*-api.ts` destiné à être consommé :

1. le contrat doit être exporté par `packages/contracts/src/api-contracts.ts` ;
2. un sous-chemin correspondant doit être déclaré dans `packages/contracts/package.json` lorsqu'un import direct est prévu ;
3. les deux points d'entrée doivent désigner le même fichier compilé ;
4. l'ajout d'un contrat ne doit pas laisser un export orphelin ou un sous-chemin absent ;
5. la documentation fonctionnelle du domaine doit référencer le contrat lorsqu'il devient une interface officielle.

Une divergence entre l'agrégat `api-contracts.ts` et la table `exports` est un défaut bloquant pour la recette, car deux applications pourraient alors voir des surfaces API différentes selon leur mode d'import.

## 12. Critères de recette

Un contrat est prêt lorsque :

- les routes et méthodes sont définies ;
- les requêtes et réponses sont typées ;
- les erreurs attendues sont cataloguées ;
- les permissions sont identifiées ;
- les règles d'idempotence sont testées ;
- les exemples ne contiennent aucune donnée réelle ;
- le point d'entrée du package exporte le contrat ;
- l'agrégat `api-contracts.ts` et les sous-chemins publiés sont cohérents ;
- le typecheck et les tests du package réussissent ;
- la documentation et le code ne présentent aucun écart connu.

## 13. Référence technique

Le catalogue technique et les conventions d'import sont décrits dans `mansa-platform/docs/contracts-catalog.md`.
