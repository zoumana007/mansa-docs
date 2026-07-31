# Messagerie Mansa Connect — Application Client

## 1. Statut et périmètre

- **Statut : Validé**
- **Application concernée : Mansa Client uniquement**
- **Nature : module financier conversationnel majeur**

La messagerie Mansa Connect ne doit pas être un simple chat. Elle doit permettre de converser, d’envoyer de l’argent, de demander de l’argent, de partager des paiements, de créer des cagnottes, de suivre les échanges financiers et d’utiliser un identifiant Mansa unique sans exposer obligatoirement son numéro de téléphone.

Le module ne doit pas être intégré tel quel dans l’application Commerçant. Les interactions commerciales restent gérées dans les modules professionnels dédiés.

## 2. Objectif produit

Créer une expérience de conversation financière différenciante dans laquelle l’utilisateur peut :

- retrouver une personne ;
- ouvrir une conversation ;
- envoyer de l’argent sans quitter le fil ;
- demander de l’argent ;
- accepter ou refuser une demande ;
- partager une addition ;
- créer une cagnotte ;
- retrouver l’historique financier de la relation ;
- utiliser son identifiant Mansa comme moyen principal de contact ;
- recevoir des notifications temps réel ;
- protéger sa vie privée.

## 3. Mansa Connect et identifiant unique

Chaque utilisateur Mansa Client possède un identifiant public unique de type :

```text
@mansa-zoumana
@camara007
```

### 3.1 Règles de l’identifiant

- Un identifiant ne peut appartenir qu’à un seul compte actif.
- Il est normalisé avant comparaison.
- Il ne doit pas contenir d’espace.
- Il doit respecter une longueur minimale et maximale configurable.
- Les caractères autorisés sont configurables, mais doivent rester simples et lisibles.
- Certains identifiants sont réservés : administration, support, banque, État, université, marques et termes sensibles.
- La fréquence de modification est limitée.
- Les anciens identifiants ne doivent pas être réattribués immédiatement.
- Toute modification est auditée.
- L’utilisateur peut choisir de ne pas être trouvable par identifiant.

### 3.2 Utilisations de l’identifiant

L’identifiant Mansa sert à :

- rechercher une personne ;
- ouvrir une conversation ;
- envoyer de l’argent ;
- demander de l’argent ;
- partager un profil ;
- inviter dans un groupe financier ;
- ajouter un bénéficiaire ;
- afficher un QR personnel ;
- partager un lien Mansa ;
- réduire la dépendance au numéro de téléphone.

## 4. Méthodes de recherche et découverte

L’utilisateur peut retrouver une personne par :

- identifiant Mansa ;
- nom et prénom, selon les paramètres de confidentialité ;
- numéro de téléphone ;
- contacts du téléphone, avec consentement explicite ;
- QR personnel ;
- lien Mansa ;
- partage NFC de profil, lorsque le téléphone et la réglementation le permettent ;
- utilisateurs récents ;
- bénéficiaires enregistrés ;
- conversations existantes.

### 4.1 Recherche par contacts

Avec l’autorisation de l’utilisateur, l’application peut comparer les contacts locaux avec les comptes Mansa.

Exemple d’affichage :

```text
18 de vos contacts utilisent Mansa
```

Aucun carnet d’adresses complet ne doit être conservé sans justification, consentement et politique de rétention explicite.

## 5. Carte de profil Mansa

Chaque utilisateur dispose d’une carte de profil partageable contenant, selon ses choix :

- photo ;
- nom affiché ;
- identifiant Mansa ;
- badge de vérification ;
- QR personnel ;
- lien partageable ;
- pays et langue, si autorisés ;
- statut de disponibilité pour recevoir des paiements ;
- bouton d’envoi d’argent ;
- bouton de demande d’argent ;
- bouton de conversation ;
- bouton de partage ;
- bouton de blocage ou signalement.

Cette carte ne doit jamais exposer le solde, l’historique financier ou des données KYC.

## 6. Types de conversations

Le module doit supporter :

- conversation individuelle ;
- groupe privé ;
- groupe financier ;
- cagnotte ;
- conversation système liée à une opération ;
- conversation avec support Mansa, dans un espace séparé et contrôlé.

Les groupes financiers doivent afficher clairement les rôles, les membres, les montants attendus, les paiements réalisés et l’objectif éventuel.

## 7. Contenu des messages

Une conversation peut contenir :

- texte ;
- emoji ;
- réactions ;
- réponse à un message ;
- image ;
- vidéo ;
- document ;
- reçu ;
- facture ;
- QR ;
- position ;
- profil Mansa ;
- demande de paiement ;
- envoi d’argent ;
- remboursement ;
- cagnotte ;
- invitation ;
- message système financier.

Les types de fichiers, tailles maximales et durées de conservation sont configurables.

## 8. Envoi d’argent dans la conversation

L’utilisateur peut appuyer sur une action `Envoyer de l’argent` depuis une conversation.

### 8.1 Parcours minimal

1. Sélectionner le montant.
2. Choisir la devise autorisée.
3. Ajouter un motif facultatif.
4. Vérifier le bénéficiaire.
5. Confirmer avec PIN, biométrie ou autre mécanisme requis.
6. Exécuter la transaction.
7. Publier une carte financière dans le fil.
8. Envoyer une notification au bénéficiaire.
9. Mettre à jour l’historique de la conversation.

### 8.2 Carte financière dans le fil

Exemple :

```text
Zoumana vous a envoyé 25 000 FCFA
Statut : Reçu
```

La carte doit afficher :

- montant ;
- devise ;
- expéditeur ;
- bénéficiaire ;
- date ;
- statut ;
- référence ;
- motif ;
- accès au reçu ;
- actions autorisées.

### 8.3 Règles métier

- L’envoi doit être idempotent.
- Le message financier ne peut pas être supprimé comme un message ordinaire.
- Le statut affiché doit provenir de la transaction réelle.
- Une transaction échouée ne doit jamais apparaître comme reçue.
- Les limites, contrôles KYC, plafonds, sanctions et règles de fraude s’appliquent.
- Les gros montants peuvent exiger une authentification renforcée.
- Un utilisateur bloqué ne peut pas initier un nouveau transfert via la conversation.

## 9. Demande d’argent

Un utilisateur peut créer une demande avec :

- montant ;
- devise ;
- motif ;
- date limite facultative ;
- pièce jointe facultative ;
- fréquence unique ou récurrente si autorisée.

Le destinataire reçoit une notification et peut :

- payer ;
- refuser ;
- proposer un autre montant ;
- demander des précisions ;
- signaler la demande.

La demande doit afficher son état :

- créée ;
- vue ;
- acceptée ;
- refusée ;
- expirée ;
- annulée ;
- payée partiellement ;
- payée totalement.

## 10. Paiement partagé

Depuis une conversation ou un groupe, l’utilisateur peut lancer `Partager l’addition`.

Le système doit permettre :

- division égale ;
- division personnalisée ;
- exclusion de certains participants ;
- ajout d’un pourboire ;
- arrondi ;
- suivi des parts payées ;
- relance manuelle ;
- rappel automatique configurable ;
- clôture de l’addition.

Chaque participant reçoit sa propre demande et paie sans quitter la conversation.

## 11. Cagnottes et groupes financiers

Une cagnotte peut servir à :

- anniversaire ;
- mariage ;
- voyage ;
- événement ;
- association ;
- aide familiale ;
- projet commun.

Elle contient :

- nom ;
- description ;
- objectif ;
- date limite ;
- devise ;
- participants ;
- rôles ;
- historique ;
- visibilité ;
- règles de retrait ;
- bénéficiaire final ;
- documents éventuels.

Les fonds ne doivent jamais être retirables hors des règles validées de la cagnotte.

## 12. Historique financier de la relation

Chaque conversation contient une vue privée organisée en onglets :

- paiements envoyés ;
- paiements reçus ;
- demandes ;
- remboursements ;
- reçus ;
- factures ;
- cagnottes ;
- médias ;
- documents.

Les totaux affichés sont calculés à partir des transactions réelles et ne doivent jamais être modifiables par le chat.

## 13. Notifications temps réel

Des notifications peuvent être envoyées pour :

- nouveau message ;
- argent reçu ;
- demande d’argent ;
- demande acceptée ;
- demande refusée ;
- paiement partagé ;
- relance ;
- cagnotte mise à jour ;
- remboursement ;
- transaction échouée ;
- activité suspecte ;
- connexion à un nouvel appareil.

L’utilisateur doit pouvoir configurer séparément les notifications de messages et les notifications financières critiques.

## 14. Confidentialité

L’utilisateur peut choisir qui peut :

- le trouver par numéro ;
- le trouver par identifiant ;
- le trouver par nom ;
- lui écrire ;
- lui envoyer une demande d’argent ;
- l’ajouter à un groupe ;
- voir sa photo ;
- voir son badge ;
- voir son statut en ligne ;
- voir les accusés de lecture.

Options possibles :

- tout le monde ;
- contacts uniquement ;
- contacts et bénéficiaires ;
- personnes autorisées ;
- personne.

## 15. Sécurité et modération

Le module doit prévoir :

- blocage ;
- signalement ;
- détection de spam ;
- limitation de débit ;
- contrôle des liens ;
- analyse des pièces jointes ;
- protection contre les demandes répétitives ;
- filtrage des comptes suspendus ;
- conservation des preuves nécessaires en cas de litige ;
- traçabilité des actions sensibles ;
- vérification renforcée avant paiement ;
- séparation claire entre message ordinaire et opération financière.

Le chiffrement des messages doit être défini selon l’architecture retenue, les obligations de support, de lutte contre la fraude et de conformité. Il ne doit pas être présenté comme chiffrement de bout en bout tant qu’il n’est pas réellement conçu, audité et compatible avec les obligations applicables.

## 16. Badges et confiance

Les profils peuvent afficher des badges vérifiés, par exemple :

- particulier vérifié ;
- professionnel ;
- banque partenaire ;
- institution publique ;
- université ;
- support officiel Mansa.

Les badges sont attribués par des workflows contrôlés et ne peuvent pas être auto-déclarés.

## 17. Jini dans la messagerie

Jini peut aider à :

- rechercher une opération ;
- résumer les paiements entre deux personnes ;
- créer un rappel ;
- expliquer un statut ;
- aider à partager une addition ;
- détecter une demande inhabituelle ;
- retrouver un reçu.

Exemple :

```text
@Jini combien ai-je envoyé à Moussa cette année ?
```

Jini ne doit jamais exécuter seul un transfert sans validation explicite de l’utilisateur.

## 18. Écrans minimums

- liste des conversations ;
- recherche universelle ;
- nouvelle conversation ;
- profil Mansa ;
- conversation individuelle ;
- groupe ;
- création de cagnotte ;
- envoi d’argent ;
- demande d’argent ;
- partage d’addition ;
- historique financier ;
- médias et documents ;
- confidentialité ;
- notifications ;
- utilisateurs bloqués ;
- signalement ;
- détails d’une transaction.

## 19. Contrats API à prévoir

Les contrats API détaillés seront documentés séparément. Le périmètre minimal couvre :

- recherche d’utilisateurs ;
- gestion de l’identifiant Mansa ;
- création et lecture de conversations ;
- envoi et lecture de messages ;
- présence et accusés de lecture ;
- pièces jointes ;
- blocage et signalement ;
- envoi d’argent ;
- demande d’argent ;
- partage d’addition ;
- groupes ;
- cagnottes ;
- notifications ;
- historique financier.

## 20. Modèle de données à prévoir

Entités minimales :

- `MansaHandle` ;
- `UserDiscoverabilitySettings` ;
- `Conversation` ;
- `ConversationMember` ;
- `Message` ;
- `MessageAttachment` ;
- `MessageReaction` ;
- `ReadReceipt` ;
- `TypingState` ou équivalent temps réel ;
- `PaymentMessageReference` ;
- `MoneyRequest` ;
- `SplitBill` ;
- `SplitBillShare` ;
- `FinancialGroup` ;
- `FundraisingPool` ;
- `BlockedUser` ;
- `UserReport` ;
- `NotificationPreference`.

Les transactions financières restent la source de vérité financière. La messagerie ne doit conserver que des références vérifiables vers ces transactions.

## 21. Critères d’acceptation

Le module est accepté lorsque :

- un utilisateur peut créer et gérer un identifiant Mansa unique ;
- la recherche respecte les paramètres de confidentialité ;
- un utilisateur peut ouvrir une conversation depuis un contact, un identifiant ou un QR ;
- un envoi d’argent est exécutable sans quitter la conversation ;
- une demande d’argent peut être acceptée ou refusée ;
- les statuts affichés correspondent aux transactions réelles ;
- les notifications financières sont reçues ;
- le partage d’addition fonctionne ;
- une cagnotte peut être créée et suivie ;
- les utilisateurs peuvent bloquer et signaler ;
- les messages financiers ne peuvent pas être falsifiés ;
- les permissions et plafonds sont appliqués ;
- les écrans sont accessibles, responsive et performants ;
- les tests unitaires, d’intégration et end-to-end critiques réussissent.

## 22. Positionnement différenciant

Mansa Connect doit se distinguer par la combinaison suivante :

- identifiant financier unique ;
- confidentialité du numéro ;
- conversation et paiement dans le même flux ;
- demandes d’argent interactives ;
- groupes financiers et cagnottes ;
- historique financier par relation ;
- profil partageable par QR, lien et NFC ;
- assistance Jini ;
- sécurité et conformité intégrées.

Le résultat attendu est une messagerie financière native, simple, sociale et utile, conçue comme une fonction centrale de l’application Mansa Client.