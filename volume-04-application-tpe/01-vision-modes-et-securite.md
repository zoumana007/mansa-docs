# Volume 4 — Application TPE : vision, modes et sécurité

## 1. Objectif

L’application TPE Mansa transforme un terminal Android compatible en poste d’encaissement professionnel. Elle doit fonctionner dans les commerces, restaurants, transports, services publics et points de vente mobiles, tout en restant administrable à distance.

## 2. Terminaux ciblés

- Terminaux Android certifiés et compatibles avec le processeur de paiement retenu.
- PAX A920 Pro ou modèle équivalent pour les démonstrations et pilotes.
- Téléphones Android en mode SoftPOS uniquement lorsque le partenaire acquéreur et les certifications l’autorisent.

La compatibilité matérielle réelle doit être validée avec le fournisseur du terminal, le SDK du processeur et les exigences PCI applicables avant production.

## 3. Modes d’environnement

### Démo

- Transactions simulées.
- Aucune donnée bancaire réelle.
- Reçus portant la mention « DÉMONSTRATION ».
- Paramètres remis à zéro sur commande administrateur.

### Recette

- Connexion aux environnements de test des partenaires.
- Cartes et moyens de paiement de test uniquement.
- Journalisation renforcée sans secrets ni données carte complètes.

### Production

- Activation explicite par l’administration centrale.
- Terminal enregistré, attribué à un commerçant et à un point de vente.
- Attestation du logiciel, clés et certificats fournis hors dépôt.
- Blocage automatique si l’intégrité, la configuration ou les certificats ne sont plus valides.

## 4. Parcours d’activation

1. L’administrateur crée le terminal dans le portail.
2. Un code d’activation à usage unique est généré.
3. Le responsable du commerce saisit ou scanne ce code sur le TPE.
4. Le backend vérifie le terminal, le commerçant, le point de vente et l’environnement.
5. Le terminal génère son identité locale sécurisée et reçoit sa configuration signée.
6. L’utilisateur s’authentifie avec son compte employé et son code local.
7. Les droits, plafonds, moyens de paiement et règles de remboursement sont synchronisés.

## 5. Encaissement standard

1. Saisie du montant ou récupération depuis le panier commerçant.
2. Présentation claire du total, des taxes, remises et pourboires éventuels.
3. Choix du moyen de paiement autorisé.
4. Création d’une intention avec clé d’idempotence.
5. Lecture ou saisie via le SDK certifié du partenaire.
6. Confirmation en ligne, ou traitement différé uniquement lorsqu’il est autorisé.
7. Affichage du résultat et génération du reçu.
8. Envoi de l’événement vers l’application commerçant et l’administration.

## 6. Sécurité obligatoire

- Aucune conservation du PAN complet, du cryptogramme ou du code PIN par Mansa.
- Utilisation exclusive des composants sécurisés et SDK certifiés pour les données carte.
- Verrouillage par code, biométrie du terminal lorsqu’elle est disponible et session courte.
- Détection du root, du débogage non autorisé, de l’altération du paquet et des versions interdites.
- Certificats et jetons propres à chaque terminal, révocables à distance.
- Configuration signée et horodatée.
- Journal d’audit local protégé puis synchronisé.
- Masquage des données sensibles dans les reçus, journaux et écrans.
- Interdiction de production sur terminal non conforme ou non attribué.

## 7. Administration à distance

L’administration peut :

- activer, suspendre ou révoquer un terminal ;
- lier le terminal à un commerce et un point de vente ;
- imposer une version minimale ;
- activer ou retirer un moyen de paiement ;
- configurer plafonds, remboursements, pourboires et impression ;
- déclencher une resynchronisation ;
- consulter l’état, la dernière connexion et les anomalies ;
- forcer la fermeture de session sans effacer les preuves d’audit.

## 8. Critères d’acceptation

- Un terminal non activé ne peut pas encaisser.
- Une commande rejouée avec la même clé d’idempotence ne crée pas un second débit.
- Le blocage distant prend effet à la prochaine communication et empêche toute nouvelle opération.
- Aucun secret réel n’est présent dans le code ou la configuration versionnée.
- Les reçus et journaux ne contiennent jamais de données carte interdites.
