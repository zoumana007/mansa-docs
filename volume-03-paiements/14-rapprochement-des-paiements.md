# Rapprochement des paiements

## Objet

Le rapprochement compare les transactions internes Mansa aux relevés reçus des banques, réseaux de cartes, opérateurs Mobile Money et processeurs. Il détecte les absences, doublons et écarts avant règlement comptable.

## Principes

- Chaque élément appartient à un lot de rapprochement.
- Au moins une référence interne ou prestataire est obligatoire.
- Tous les montants utilisent l’unité monétaire mineure et des entiers sûrs.
- La devise est normalisée sur trois lettres.
- Un rapprochement automatique n’altère jamais le grand livre.
- Toute résolution manuelle exige une justification et un audit.
- Les fichiers partenaires bruts sont conservés dans un stockage sécurisé avec empreinte et politique de rétention.

## États

- `PENDING` : élément créé mais comparaison non terminée.
- `MATCHED` : références et montant concordent.
- `PARTIALLY_MATCHED` : rapprochement partiel nécessitant une analyse.
- `MISMATCHED` : écart confirmé.
- `RESOLVED` : écart traité et justifié.
- `IGNORED` : écart explicitement exclu avec justification.

Les états finaux sont `MATCHED`, `RESOLVED` et `IGNORED`.

## Motifs d’écart

- `MISSING_INTERNAL_TRANSACTION` ;
- `MISSING_PROVIDER_TRANSACTION` ;
- `AMOUNT_MISMATCH` ;
- `CURRENCY_MISMATCH` ;
- `STATUS_MISMATCH` ;
- `DUPLICATE_PROVIDER_TRANSACTION` ;
- `OTHER`.

## Règles automatiques initiales

1. Si la référence interne manque, l’élément est `MISMATCHED` avec `MISSING_INTERNAL_TRANSACTION`.
2. Si la référence prestataire manque, il est `MISMATCHED` avec `MISSING_PROVIDER_TRANSACTION`.
3. Si les montants diffèrent, il est `MISMATCHED` avec `AMOUNT_MISMATCH`.
4. Si les deux références et montants concordent, il est `MATCHED`.
5. Les contrôles de devise, statut et doublon seront enrichis avec les adaptateurs partenaires et les règles de lot.

## Traitement opérationnel

Le traitement quotidien doit :

1. importer le fichier ou flux partenaire ;
2. vérifier son identité, son format, son empreinte et sa période ;
3. associer chaque ligne à une transaction interne ;
4. produire les éléments de rapprochement ;
5. isoler les écarts ;
6. affecter les écarts aux équipes compétentes ;
7. enregistrer les résolutions ;
8. produire un rapport de clôture de lot.

Une résolution ne doit jamais modifier directement un solde. Toute correction financière passe par une écriture de grand livre autorisée, idempotente et liée à l’élément de rapprochement.

## Sécurité et séparation des tâches

L’import, l’analyse et l’approbation des corrections doivent pouvoir être attribués à des rôles distincts. Les fichiers partenaires peuvent contenir des données sensibles : ils ne doivent pas être enregistrés dans les logs applicatifs ni dans Git.

Les résolutions importantes doivent utiliser une approbation à quatre yeux selon les seuils configurés.

## Observabilité

Les métriques minimales sont :

- nombre et montant des transactions rapprochées ;
- nombre et montant des écarts par partenaire et motif ;
- âge moyen et maximal des écarts ouverts ;
- taux de rapprochement automatique ;
- lots incomplets ou reçus en retard ;
- résolutions manuelles et corrections financières associées.

## Alignement avec le code

Le contrat de référence est `packages/contracts/src/reconciliation.ts` dans `mansa-platform`.

Il expose :

- `RECONCILIATION_STATUSES` et `RECONCILIATION_MISMATCH_REASONS` ;
- `ReconciliationItem` et les commandes associées ;
- `createReconciliationItem` ;
- `resolveReconciliationItem` ;
- les gardes de type et `isFinalReconciliationStatus` ;
- le sous-chemin public `@mansa/contracts/reconciliation`.

## Critères d’acceptation

- Une ligne sans aucune référence est rejetée.
- Les montants négatifs ou non entiers sont rejetés.
- Une devise invalide est rejetée.
- Une concordance exacte produit `MATCHED`.
- Une absence de référence ou un écart de montant produit `MISMATCHED` avec le motif exact.
- Seuls les écarts ouverts peuvent être résolus ou ignorés.
- Toute résolution contient une justification non vide.
- Les tests couvrent concordance, absence, écart, résolution et données invalides.
- La documentation et les exports publics restent synchronisés.
