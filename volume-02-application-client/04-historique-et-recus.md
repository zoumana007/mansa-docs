# Historique des transactions et reçus

## Objectif

L’historique donne au client une vue fiable, filtrable et compréhensible de toutes les opérations liées à ses portefeuilles. Il couvre les transferts, paiements, encaissements, retraits, dépôts, remboursements, frais et opérations de service public sans exposer de données sensibles.

## Contenu d’une ligne

Chaque élément affiche au minimum :

- le type d’opération ;
- le sens `INCOMING`, `OUTGOING` ou `NEUTRAL` ;
- le montant et la devise ;
- le statut ;
- la contrepartie ou le libellé utile ;
- la date de création ;
- une référence consultable.

Les numéros de compte, téléphone, carte et identifiants partenaires restent masqués. Le détail complet n’est chargé qu’après sélection de l’opération.

## Filtres et recherche

Le client peut filtrer par portefeuille, type, statut, sens, devise, plage de montant et plage de dates. Une recherche textuelle peut porter sur une référence, un libellé ou une contrepartie autorisée. Le tri est proposé par date ou montant, en ordre croissant ou décroissant.

La pagination doit être stable : l’ajout d’une nouvelle transaction ne doit pas provoquer de doublons lors du chargement de la page suivante. Le backend utilise le contrat commun de pagination et limite la taille maximale d’une page.

## Détail d’une transaction

La vue détail présente :

- la référence Mansa ;
- le montant principal et les frais séparés ;
- le statut courant ;
- les dates de création et de finalisation ;
- la contrepartie masquée ;
- le canal ou moyen de paiement ;
- les informations utiles au support ;
- les actions autorisées : télécharger le reçu, partager, répéter ou signaler un problème.

Une opération en attente ne doit jamais être présentée comme définitive. Une opération annulée, échouée ou remboursée conserve son historique et son statut réel.

## Reçus

Un reçu est produit à partir des données comptabilisées par le backend, jamais à partir des seules données affichées par le mobile. Il contient un numéro unique, la référence de transaction, la date d’émission, le montant, les frais, la devise, le statut et les informations légales configurées pour le pays et le partenaire.

Le reçu peut être affiché, téléchargé ou partagé. Le document partagé ne contient aucun jeton, identifiant interne, numéro complet de carte ou donnée KYC. Un reçu ne constitue pas une preuve de règlement définitif tant que la transaction n’est pas dans un état final réussi.

## Règles métier

- Les montants utilisent le type partagé `Money`.
- Les filtres de montant utilisent les unités mineures sous forme de chaîne et non des nombres flottants.
- Les dates sont transmises au format ISO 8601 avec fuseau explicite.
- Les résultats sont limités aux portefeuilles accessibles par la session active.
- Les données sensibles sont masquées côté backend avant livraison au client.
- Toute consultation inhabituelle ou export massif peut déclencher un contrôle de risque.
- Les métadonnées libres d’un reçu sont limitées à des chaînes non sensibles autorisées.
- Les opérations supprimées fonctionnellement restent conservées dans les journaux financiers selon les règles de rétention.

## Contrats techniques alignés

Le dépôt `mansa-platform` expose `packages/contracts/src/transaction-history.ts` et le sous-chemin `@mansa/contracts/transaction-history`. Les contrats couvrent `TransactionHistoryItem`, `TransactionHistoryFilter`, `ListTransactionHistoryQuery`, `TransactionHistoryPage`, `TransactionReceipt`, les catalogues de direction et de tri ainsi que leurs gardes de type.

## Critères d’acceptation

1. Le client ne voit que les transactions de ses portefeuilles autorisés.
2. Les filtres combinés retournent des résultats cohérents et paginés sans doublon.
3. Les montants et frais du détail correspondent aux écritures comptabilisées.
4. Une transaction non finalisée est clairement identifiée comme en attente.
5. Un reçu ne révèle aucune donnée complète de carte, de compte ou de téléphone.
6. Les reçus d’opérations réussies possèdent un numéro unique et une référence vérifiable.
7. Les valeurs de direction et de tri sont couvertes par des tests sans doublons.
