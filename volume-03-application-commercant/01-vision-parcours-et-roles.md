# Volume 3 — Application Commerçant : vision, parcours et rôles

## 1. Objectif

L’application Commerçant permet à une entreprise, un indépendant ou une organisation d’encaisser, gérer ses points de vente, suivre ses flux, administrer ses employés et exploiter des services commerciaux sans dépendre en permanence d’un terminal physique.

## 2. Profils utilisateurs

- **Propriétaire** : contrôle complet du commerce, des établissements, des comptes de règlement et des habilitations.
- **Gestionnaire** : supervise les opérations, remboursements, équipes, catalogues et rapports selon délégation.
- **Caissier** : encaisse, consulte ses opérations et effectue uniquement les actions autorisées.
- **Comptable** : consulte et exporte les transactions, rapprochements, frais et règlements.
- **Support interne** : consulte les incidents et prépare les informations nécessaires à leur résolution.

Chaque rôle repose sur des permissions granulaires. Un rôle personnalisé peut être créé par l’administration ou le propriétaire lorsque cette fonction est autorisée.

## 3. Inscription commerçant

Le parcours comprend :

1. création ou rattachement d’une identité Mansa ;
2. saisie du type d’activité et des informations légales ;
3. déclaration des bénéficiaires effectifs lorsque nécessaire ;
4. ajout des pièces justificatives ;
5. sélection du compte ou portefeuille de règlement ;
6. revue de conformité ;
7. activation du commerce et de ses fonctionnalités.

L’état du dossier doit être visible : brouillon, soumis, à compléter, en revue, approuvé, rejeté, suspendu.

## 4. Organisation commerciale

Un compte commerçant peut contenir plusieurs :

- enseignes ;
- établissements ;
- points de vente ;
- terminaux ;
- employés ;
- catalogues ;
- comptes de règlement.

Toutes les données doivent être filtrables par établissement et période. Les changements de propriétaire, compte de règlement ou statut exigent une authentification renforcée et un audit.

## 5. Encaissement

L’application doit permettre :

- QR statique et dynamique ;
- lien ou demande de paiement ;
- saisie manuelle d’un montant ;
- panier produit ;
- paiement fractionné entre plusieurs clients ;
- pourboire optionnel ;
- référence de commande ;
- reçu numérique ;
- paiement via TPE associé ;
- mode hors ligne limité lorsque le risque et le matériel l’autorisent.

Le montant, la devise, les frais éventuels et l’identité du commerce sont toujours affichés avant validation.

## 6. Transactions et après-vente

Le commerçant consulte les paiements en attente, réussis, échoués, expirés, annulés, remboursés ou contestés.

Selon les permissions, il peut :

- rembourser totalement ou partiellement ;
- annuler une intention non finalisée ;
- renvoyer un reçu ;
- ajouter une note interne ;
- rechercher par référence, client masqué, terminal ou employé ;
- ouvrir un dossier support ;
- suivre une contestation.

Aucun remboursement ne modifie l’opération d’origine : il crée une transaction compensatoire liée.

## 7. Règlements et frais

Le tableau de bord distingue :

- chiffre d’affaires brut ;
- remboursements ;
- frais Mansa ;
- taxes ou retenues configurées ;
- montant net ;
- sommes en attente ;
- prochains règlements ;
- anomalies de rapprochement.

Les règles de règlement sont configurables par pays, partenaire, type de commerce, niveau de risque et contrat.

## 8. Employés et sécurité

- Invitation par téléphone ou e-mail vérifié.
- Affectation à un ou plusieurs établissements.
- Permissions minimales par défaut.
- Révocation immédiate des sessions lors d’un retrait d’accès.
- Authentification renforcée pour remboursements, exports sensibles et changements de règlement.
- Journal d’audit consultable selon permission.

## 9. Expérience et disponibilité

L’interface doit rester simple pour un petit commerce tout en supportant les besoins d’un réseau multi-sites. Les fonctions non activées par contrat, pays ou conformité ne doivent pas être affichées comme disponibles.

## 10. Fidélité et récompenses

Chaque commerçant peut activer un programme de fidélité distinct. Le programme définit le nom des points, le taux d’acquisition, le seuil minimal de dépense, la durée de validité et les récompenses disponibles.

Règles obligatoires :

- un compte de fidélité est unique par client et programme ;
- les points sont des entiers et ne constituent pas une monnaie électronique ;
- chaque acquisition, utilisation, ajustement, expiration ou annulation produit une écriture immuable ;
- toute commande d’acquisition ou d’utilisation possède une clé d’idempotence et une référence métier ;
- un solde ne peut jamais devenir négatif ;
- une annulation crée une écriture inverse liée à l’écriture d’origine ;
- un ajustement manuel exige un code motif, une justification, une permission dédiée et un audit ;
- les points expirés ne peuvent pas être réactivés sans opération administrative explicite ;
- la disponibilité d’une récompense tient compte de son statut, de sa période, de son stock éventuel et du nombre maximal d’utilisations ;
- les conditions visibles par le client doivent correspondre à la configuration réellement appliquée.

Les statuts du programme sont `DRAFT`, `ACTIVE`, `PAUSED` et `ARCHIVED`. Les comptes clients sont `ACTIVE`, `SUSPENDED` ou `CLOSED`. Les mouvements sont `EARN`, `REDEEM`, `ADJUST`, `EXPIRE` ou `REVERSE`.

Le socle de contrats correspondant est défini dans `mansa-platform/packages/contracts/src/merchant.ts` avec les modèles `LoyaltyProgram`, `LoyaltyAccount`, `LoyaltyTransaction`, `LoyaltyReward` et les commandes d’acquisition, d’utilisation et d’ajustement.

## 11. Critères d’acceptation

- Un propriétaire peut créer un établissement et inviter un caissier sans intervention technique.
- Un caissier ne peut ni modifier le compte de règlement ni consulter les données d’autres établissements sans permission.
- Chaque encaissement possède une référence unique et un état explicite.
- Les remboursements sont traçables et liés à l’opération d’origine.
- Les totaux du tableau de bord sont réconciliables avec les exports transactionnels.
- Deux demandes identiques d’acquisition de points avec la même clé d’idempotence ne créditent qu’une seule fois le compte.
- Une utilisation de récompense est refusée lorsque le solde est insuffisant ou que la récompense est inactive, expirée ou épuisée.
- Une annulation de mouvement conserve l’historique complet et produit une écriture inverse.
- Un ajustement manuel sans motif, justification ou permission est refusé et audité.
