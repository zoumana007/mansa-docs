# Volume 3 — Encaissements, remboursements et règlements

## 1. Intention de paiement

Toute opération commence par une intention de paiement contenant au minimum : identifiant du commerçant, établissement, montant en unités mineures, devise, canal, référence métier, expiration et clé d’idempotence.

Une intention ne débite pas automatiquement le client. Elle représente l’état contrôlé du parcours jusqu’à confirmation, expiration ou annulation.

## 2. Canaux supportés

- QR dynamique généré pour une commande précise.
- QR statique rattaché à un commerce ou point de vente.
- Demande de paiement envoyée à un client Mansa.
- Lien de paiement à durée limitée.
- Terminal TPE associé.
- Paiement initié depuis un panier de produits.

Chaque canal doit converger vers le même modèle transactionnel et les mêmes règles d’audit.

## 3. Paiement partagé

Un panier peut être réparti :

- à parts égales ;
- par montant personnalisé ;
- par article ;
- entre plusieurs moyens de paiement autorisés.

Le système conserve le montant total attendu, chaque contribution, le solde restant et l’expiration. La commande n’est marquée payée que lorsque la somme des contributions finalisées atteint exactement le total, sauf tolérance explicitement configurée.

## 4. Pourboires et remises

Les pourboires sont séparés du prix principal dans les données et reçus. Les remises doivent préciser leur origine : promotion, coupon, geste commercial ou programme de fidélité.

Les limites, bénéficiaires et traitements comptables sont configurables par marché.

## 5. Remboursements

Un remboursement :

- référence obligatoirement un paiement finalisé ;
- possède son propre identifiant et sa propre clé d’idempotence ;
- ne peut dépasser le montant remboursable restant ;
- peut nécessiter une authentification renforcée ;
- enregistre l’auteur, le motif, l’établissement et le terminal ;
- crée des écritures compensatoires dans le grand livre.

Les règles définissent qui peut initier ou approuver un remboursement selon le montant.

## 6. Contestations

Une contestation possède un cycle de vie indépendant : ouverte, preuves requises, preuves soumises, en revue, gagnée, perdue, clôturée.

Le commerçant reçoit les échéances, motifs, montants exposés et pièces demandées. Les fonds bloqués ou débités sont visibles séparément des ventes disponibles.

## 7. Règlement commerçant

Le règlement agrège les opérations d’une période et calcule :

`net = ventes brutes - remboursements - frais + ajustements - retenues`

Chaque élément du calcul doit être explicable par une ligne source. Les règlements peuvent être quotidiens, hebdomadaires, manuels ou déclenchés selon un seuil.

États : planifié, en traitement, payé, échoué, retenu, annulé.

## 8. Rapprochement

Le système rapproche :

- paiements internes ;
- confirmations des partenaires ;
- écritures du grand livre ;
- règlements bancaires ou Mobile Money ;
- remboursements et ajustements.

Toute différence crée une anomalie attribuable, horodatée et suivie jusqu’à résolution.

## 9. Reçus et justificatifs

Un reçu contient une référence vérifiable, la date, le commerce, le point de vente, le montant, la devise, les taxes configurées, le canal, le statut et les informations client strictement nécessaires.

Les données de carte complètes, secrets, codes ou documents d’identité ne doivent jamais apparaître.

## 10. Exigences de contrôle

- Idempotence obligatoire pour création, confirmation, annulation et remboursement.
- Aucun montant flottant.
- États finaux immuables ; toute correction passe par compensation.
- Double validation configurable au-delà d’un seuil.
- Limitation de débit et détection des comportements anormaux.
- Journal d’audit pour chaque décision sensible.

## 11. Critères d’acceptation

- Deux requêtes identiques avec la même clé d’idempotence ne créent pas deux paiements.
- Le cumul des remboursements ne dépasse jamais le paiement d’origine.
- Le détail d’un règlement permet de reconstruire son montant net.
- Une anomalie partenaire ne modifie pas silencieusement un solde.
- Les données affichées au commerçant respectent ses établissements et permissions.
