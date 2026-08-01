# Cartes physiques, virtuelles et jetables

## 1. Objectif

Mansa Client doit permettre à un utilisateur éligible de demander, activer, consulter, sécuriser et utiliser une carte physique ou virtuelle sans exposer les données sensibles dans l’application, les journaux ou les reçus.

Les cartes sont proposées uniquement lorsqu’un émetteur, un processeur et les autorisations réglementaires nécessaires sont disponibles dans le pays concerné.

## 2. Types de cartes

La plateforme peut gérer :

- carte physique ;
- carte virtuelle permanente ;
- carte virtuelle temporaire ;
- carte jetable renouvelée après chaque paiement éligible ;
- carte dédiée à un budget, un projet ou une équipe ;
- carte employé rattachée à une entreprise ;
- carte de démonstration sans accès aux réseaux réels.

Chaque type est activé par pays, partenaire, devise, niveau KYC, segment client et environnement.

## 3. États communs

Les états minimums sont :

- `pending` : créée mais non activée ;
- `active` : utilisable ;
- `frozen` : gelée temporairement par le client ou l’administration ;
- `blocked` : bloquée pour sécurité, conformité ou fraude ;
- `expired` : arrivée à échéance ;
- `terminated` : clôturée définitivement.

Le dépôt `mansa-platform` fournit le contrat de domaine `PaymentCard` afin que les applications Client, Admin et TPE partagent les mêmes transitions.

## 4. Demande d’une carte physique

Le parcours vérifie avant commande :

- identité et niveau KYC ;
- pays et adresse de livraison ;
- disponibilité du partenaire ;
- coût de fabrication et de livraison ;
- limites applicables ;
- acceptation des conditions ;
- risque et éventuels contrôles renforcés.

L’utilisateur confirme le nom imprimé, l’adresse, les frais et le délai indicatif. Une commande possède une référence indépendante de la carte afin de suivre fabrication, expédition, livraison, retour ou annulation.

## 5. Activation

Une carte physique est activée uniquement après une preuve prévue par le partenaire, par exemple :

- saisie d’un code reçu séparément ;
- lecture d’informations partielles ;
- première opération avec code PIN ;
- confirmation renforcée dans l’application.

Une carte virtuelle peut être active immédiatement après validation du produit ou rester en attente jusqu’à authentification forte.

Une carte expirée, bloquée ou clôturée ne peut pas être activée.

## 6. Données affichées

L’écran principal affiche uniquement les données nécessaires :

- nom du produit ;
- type de carte ;
- réseau ;
- quatre derniers chiffres ;
- état ;
- date d’expiration partiellement masquée selon le contexte ;
- portefeuille ou compte rattaché ;
- plafonds ;
- options de sécurité ;
- opérations récentes.

Le PAN complet, le cryptogramme et le PIN ne sont jamais stockés en clair dans le backend Mansa. Leur affichage éventuel repose sur un composant sécurisé du processeur, une authentification forte, une durée courte et l’absence de capture dans les journaux.

## 7. Gel et dégel

L’utilisateur peut geler immédiatement une carte active. Le gel doit :

- empêcher les nouvelles autorisations selon les capacités du processeur ;
- conserver l’historique ;
- rester réversible ;
- être propagé au partenaire ;
- produire un événement et une trace d’audit ;
- afficher clairement les opérations qui peuvent encore être présentées tardivement.

Le dégel exige une carte encore valide et une authentification adaptée au risque.

## 8. Blocage et clôture

Le blocage est utilisé pour perte, vol, suspicion de fraude, décision de conformité ou demande d’un partenaire. Il ne doit pas être confondu avec un simple gel temporaire.

La clôture est définitive. Elle doit prévoir :

- traitement des autorisations en attente ;
- conservation des preuves comptables ;
- remboursement ou transfert du solde associé lorsque nécessaire ;
- désactivation des jetons Wallet ;
- règles de remplacement ;
- notification au client ;
- journal d’audit.

## 9. PIN

Les fonctions possibles sont :

- choix ou récupération contrôlée du PIN ;
- changement du PIN ;
- déblocage après échecs lorsque le partenaire l’autorise ;
- consultation sécurisée et limitée ;
- rappel des règles de confidentialité.

Mansa ne doit jamais envoyer un PIN complet par e-mail, SMS ou notification push.

## 10. Contrôles configurables

Le client peut, selon le produit, activer ou désactiver :

- paiements en ligne ;
- paiements sans contact ;
- retraits ;
- paiements à l’étranger ;
- paiements par bande magnétique ;
- catégories de commerçants ;
- pays ou zones ;
- plafonds par opération, jour ou mois ;
- notifications instantanées.

Les réglages sont validés côté serveur et répercutés chez le processeur. L’application ne doit jamais afficher un réglage comme actif avant confirmation de la source d’autorité.

## 11. Carte virtuelle jetable

Une carte jetable utilise des références renouvelées après une autorisation réussie ou selon une règle du partenaire.

Le système doit distinguer :

- l’identité stable du produit carte ;
- le jeton ou numéro temporaire ;
- la transaction ayant provoqué la rotation ;
- les autorisations différées ;
- les remboursements liés à un ancien numéro ;
- les commerçants nécessitant des paiements récurrents.

La carte jetable est désactivée automatiquement pour les usages incompatibles, notamment certains abonnements, cautions, hôtels, locations ou paiements hors ligne.

## 12. Wallets mobiles

Une carte éligible peut être ajoutée à Apple Wallet, Google Wallet ou un portefeuille partenaire par tokenisation.

La plateforme conserve uniquement les références et états nécessaires :

- demande de tokenisation ;
- appareil ;
- état du jeton ;
- suspension ;
- reprise ;
- suppression ;
- fournisseur de tokenisation.

La suppression d’une carte dans Mansa et la suppression d’un jeton sur un appareil sont deux opérations distinctes mais coordonnées.

## 13. Autorisations et opérations

Pour chaque tentative, l’utilisateur peut voir :

- commerçant ;
- montant demandé ;
- devise ;
- montant comptabilisé ;
- taux et frais ;
- état de l’autorisation ;
- date ;
- carte masquée ;
- lieu approximatif lorsque disponible ;
- référence ;
- action de contestation.

Les états doivent distinguer autorisation, capture, annulation, expiration, remboursement, rejet et rétrofacturation.

## 14. Sécurité et fraude

Les contrôles minimums comprennent :

- authentification forte pour les actions sensibles ;
- données carte masquées ;
- chiffrement et tokenisation ;
- détection de changement d’appareil ;
- limitation des consultations sensibles ;
- alertes en temps réel ;
- règles de vélocité ;
- géographie et catégories ;
- blocage automatique configurable ;
- procédure de contestation ;
- séparation stricte Démo, Recette et Production.

Aucun secret processeur, numéro de carte réel, cryptogramme ou PIN ne doit apparaître dans les dépôts.

## 15. Administration

L’administration peut configurer :

- produits et types de cartes ;
- pays, devises et partenaires ;
- prix, livraison et remplacement ;
- niveaux KYC ;
- plafonds ;
- usages autorisés ;
- règles de gel, blocage et clôture ;
- cartes jetables ;
- tokenisation ;
- notifications ;
- maintenance ;
- motifs de refus ;
- matrices de permissions.

Chaque changement est versionné et audité.

## 16. Critères de recette

- Une carte est créée en état `pending` sauf règle explicite validée.
- Une carte en attente peut être activée avant son expiration.
- Une carte active peut être gelée puis dégelée.
- Une carte gelée, bloquée, expirée ou clôturée ne peut pas autoriser de nouvelle opération.
- Une carte bloquée ou clôturée ne peut pas être réactivée par le client.
- Une carte ne peut pas expirer avant sa date d’échéance.
- Seuls les quatre derniers chiffres sont présents dans le contrat métier Mansa.
- Les dates internes ne peuvent pas être modifiées par une référence externe.
- Les données sensibles ne sont jamais écrites dans les journaux.
- Une modification de contrôle n’est confirmée qu’après accusé du processeur.
- Toute action sensible produit un événement et une trace d’audit.
- Les états affichés dans Client et Admin sont cohérents avec le backend.
