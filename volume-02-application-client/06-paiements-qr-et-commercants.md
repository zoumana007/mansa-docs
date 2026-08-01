# Paiements QR et paiements commerçants

## 1. Objectif

Mansa Client doit permettre de payer un commerçant rapidement par QR, lien de paiement, identifiant commerçant ou terminal compatible, sans sacrifier la confirmation du bénéficiaire, la transparence des frais ni la traçabilité comptable.

Le paiement commerçant est une opération financière distincte d’un simple transfert entre particuliers. Il porte un contexte commercial, un bénéficiaire professionnel, une référence de vente, des règles de remboursement et, lorsque nécessaire, une preuve de livraison ou de prestation.

## 2. Modes de paiement

L’application peut proposer :

- scan d’un QR statique ;
- scan d’un QR dynamique ;
- affichage d’un QR client présenté au commerçant ;
- paiement depuis un lien ;
- recherche d’un commerçant dans l’annuaire ;
- paiement depuis une facture ou une commande ;
- paiement initié par un terminal TPE ;
- paiement sans contact lorsque le téléphone et le partenaire le permettent ;
- paiement récurrent explicitement autorisé ;
- paiement partagé entre plusieurs personnes.

Chaque mode est activé par pays, devise, commerçant, catégorie, niveau KYC, risque, environnement et partenaire de paiement.

## 3. QR statique

Un QR statique identifie principalement un commerçant ou un point de vente. Le client saisit le montant après le scan.

Le backend doit résoudre le QR et retourner uniquement les données nécessaires :

- identité commerciale ;
- nom du point de vente ;
- identifiant commerçant ;
- catégorie ;
- devise acceptée ;
- moyens de paiement disponibles ;
- statut actif ou indisponible ;
- avertissement éventuel.

Le montant saisi doit être strictement positif et validé par les règles de limites, solde, conformité et fraude.

## 4. QR dynamique

Un QR dynamique représente une intention de paiement limitée dans le temps. Il contient ou référence :

- un identifiant unique ;
- un commerçant ;
- un montant fixe ou libre ;
- une devise ;
- une référence de commande ;
- une date d’expiration ;
- un statut ;
- des métadonnées signées ou récupérées côté serveur.

Le QR ne doit pas constituer seul la source d’autorité. L’application transmet son identifiant au backend, qui vérifie l’intégrité, l’état, l’expiration et le commerçant avant d’afficher la confirmation.

## 5. États d’une intention QR

Les états minimums sont :

- `active` ;
- `consumed` ;
- `expired` ;
- `cancelled`.

Une intention consommée, expirée ou annulée ne peut plus être payée. Une intention active ne peut être consommée qu’une seule fois, avec protection par idempotence.

Le dépôt `mansa-platform` fournit le contrat de domaine `QrPaymentIntent` pour représenter ce cycle de vie.

## 6. Confirmation avant paiement

Avant authentification finale, l’écran affiche :

- commerçant et point de vente ;
- montant ;
- devise ;
- frais éventuels ;
- taxes visibles lorsqu’elles sont applicables ;
- total débité ;
- source de financement ;
- référence de commande ;
- avantages ou fidélité ;
- conditions de remboursement ;
- avertissement de risque éventuel.

Toute modification du montant, du commerçant, des frais, du taux ou de la source de financement invalide la confirmation précédente.

## 7. Source de financement

Le paiement peut débiter :

- un portefeuille Mansa ;
- un compte bancaire partenaire ;
- une carte enregistrée ;
- un solde promotionnel autorisé ;
- un portefeuille Mobile Money ;
- une combinaison de sources, lorsque le produit l’autorise.

Le backend reste responsable de la sélection finale, des contrôles, des réservations et de l’écriture comptable.

## 8. Exécution et idempotence

Chaque tentative logique possède une clé d’idempotence stable. La répétition de la requête ne doit jamais produire un second débit.

Pour un paiement interne, le débit client, le crédit commerçant, les frais, commissions et taxes sont enregistrés de manière équilibrée.

Pour un paiement externe, le système distingue :

1. validation de l’intention ;
2. réservation des fonds ;
3. envoi au partenaire ;
4. accusé de réception ;
5. confirmation finale ;
6. échec ou expiration ;
7. libération ou compensation.

Le statut « réussi » n’est présenté qu’après confirmation de la source d’autorité.

## 9. Reçu commerçant

Le reçu contient au minimum :

- référence Mansa ;
- référence commerçant ;
- date et heure ;
- identité commerciale ;
- point de vente ;
- montant ;
- devise ;
- frais ;
- taxes ;
- source de paiement masquée ;
- statut ;
- identifiant de commande ;
- informations de remboursement.

Le reçu partageable masque les données sensibles inutiles.

## 10. Remboursement

Un remboursement ne supprime ni ne modifie l’opération originale. Il crée une nouvelle opération liée à la précédente.

Les remboursements peuvent être :

- complets ;
- partiels ;
- initiés par le commerçant ;
- demandés par le client ;
- soumis à validation ;
- exécutés automatiquement après annulation d’une réservation ;
- refusés avec un motif traçable.

Le cumul des remboursements ne peut pas dépasser le montant remboursable de l’opération originale.

## 11. Paiement partagé

Pour une facture collective, le commerçant peut créer une intention contenant un total et plusieurs parts. Les clients peuvent payer :

- une part égale ;
- un montant personnalisé ;
- des articles précis ;
- un pourboire ;
- une part restante.

La somme encaissée ne doit pas dépasser le montant autorisé, sauf règle explicite pour pourboire. La facture se ferme automatiquement lorsque le total attendu est atteint.

## 12. Fidélité et promotions

Une offre peut appliquer :

- remise immédiate ;
- cashback ;
- points ;
- coupon ;
- gratuité de frais ;
- avantage financé par Mansa, le commerçant ou un partenaire.

L’avantage est calculé côté serveur, versionné et visible avant validation. Une promotion ne doit jamais déséquilibrer les écritures comptables.

## 13. Sécurité et fraude

Le système doit prévoir :

- validation cryptographique ou résolution serveur du QR ;
- détection des QR falsifiés ou expirés ;
- confirmation renforcée pour montant inhabituel ;
- contrôle de proximité lorsque pertinent ;
- détection de répétitions rapides ;
- contrôle du commerçant et du terminal ;
- blocage par catégorie ou pays ;
- limitation des nouveaux appareils ;
- authentification forte selon le risque ;
- signalement d’un paiement suspect.

Les motifs internes détaillés de fraude ne sont pas révélés au client.

## 14. Fonctionnement hors ligne

Le client ne doit pas considérer un paiement hors ligne comme définitivement réussi sans preuve autorisée.

Lorsque le produit prend en charge un mode dégradé :

- les plafonds sont faibles et configurables ;
- le terminal ou l’appareil possède une autorisation limitée ;
- les jetons sont signés et à usage contrôlé ;
- la synchronisation est obligatoire ;
- les conflits et doubles dépenses sont détectés ;
- le risque financier est attribué explicitement.

## 15. Administration

L’administration peut configurer :

- types de QR ;
- durée de validité ;
- montants minimums et maximums ;
- devises ;
- catégories autorisées ;
- commissions ;
- frais client et commerçant ;
- remboursements ;
- pourboires ;
- promotions ;
- fidélité ;
- authentification forte ;
- mode hors ligne ;
- maintenance par partenaire ou point de vente.

Toute modification est versionnée, datée et auditée.

## 16. Contrats techniques

Le socle technique doit conserver les responsabilités suivantes :

- `QrPaymentIntent` : cycle de vie de l’intention QR ;
- `Money` : montant en unités mineures ;
- `TransactionLimitPolicy` : limites ;
- `FeePolicy` : frais ;
- `TransactionService` et grand livre : exécution comptable ;
- contrats d’idempotence : prévention des doubles débits ;
- outbox et événements : notifications et intégrations fiables.

Les applications Client, Commerçant et TPE doivent partager les mêmes états et références métier.

## 17. Critères de recette

- Un QR inconnu ou invalide est refusé.
- Un QR expiré, annulé ou déjà consommé ne peut pas être payé.
- Un QR dynamique actif ne peut être consommé qu’une fois.
- Le commerçant est résolu côté serveur avant confirmation.
- Un montant fixe ne peut pas être remplacé par le client.
- Un montant libre doit être strictement positif.
- La devise du paiement correspond à celle autorisée.
- Les frais et le total débité sont affichés avant validation.
- Une même clé d’idempotence ne crée jamais deux débits.
- Le paiement réussi produit des écritures équilibrées.
- Le reçu contient les références client et commerçant utiles.
- Le cumul des remboursements ne dépasse jamais le montant remboursable.
- Une promotion est appliquée selon une règle versionnée.
- Les actions sensibles et transitions d’état sont auditées.
- Les applications Client, Commerçant, TPE et Admin affichent des statuts cohérents.
