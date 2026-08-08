# Bornes de péage Mansa — paiement multicanal et adaptation véhicules

## 1. Objet

Cette spécification complète le moteur `Access & Mobility` pour les voies de péage équipées de bornes physiques. Elle couvre la borne conducteur, les moyens de paiement, le QR Mansa, l’acceptation des espèces, la détection de billets/pièces non conformes, les poids lourds, les pannes et la continuité de service.

Le logiciel Mansa doit rester indépendant d’un fabricant unique. Les validateurs de billets, monnayeurs, TPE, scanners, imprimantes, interphones, barrières et capteurs sont intégrés par adaptateurs matériels.

## 2. Borne adaptée aux véhicules

Une voie mixte voitures/camions doit privilégier une borne `DUAL_HEIGHT` :

- interface basse pour voiture et véhicule léger ;
- interface haute pour camion, bus et cabine élevée ;
- les deux interfaces pilotent la même transaction de voie ;
- l’état de la voie et le montant dû sont identiques sur les deux niveaux ;
- les composants peuvent être doublés physiquement ou mutualisés selon la conception matérielle.

Profils supportés par le contrat partagé :

```text
SINGLE_LOW
SINGLE_HIGH
DUAL_HEIGHT
```

Le profil de hauteur est une caractéristique du terminal, pas une règle tarifaire.

## 3. Ordre d’interaction au péage

Le RFID n’est pas présenté comme un bouton de paiement.

Ordre cible :

1. le véhicule entre dans la zone de détection ;
2. le système tente automatiquement RFID UHF et lecture de plaque lorsque la voie le permet ;
3. si un abonnement/droit valide est trouvé, le passage est traité sans demander au conducteur de choisir « RFID » ;
4. si aucun droit automatique valide n’est trouvé, la borne affiche le montant ;
5. elle propose uniquement les moyens de paiement réellement disponibles ;
6. le paiement est confirmé ;
7. la barrière s’ouvre ou le passage est enregistré ;
8. le reçu et l’audit sont générés selon la politique.

Exemple écran non-abonné :

```text
Montant à payer : 1 000 FCFA

Choisissez votre moyen de paiement :
[ Carte ] [ Mobile Money ] [ QR Mansa ] [ Billets ] [ Pièces ]
```

La liste doit être calculée depuis `AccessServiceAvailability.availablePaymentMethods` et ne doit jamais afficher un moyen indisponible.

## 4. QR Mansa

Deux modes sont supportés :

```text
TERMINAL_SCANS_CUSTOMER_QR
CUSTOMER_SCANS_DYNAMIC_TERMINAL_QR
```

### 4.1 La borne scanne le QR du client

Le client ouvre l’application Mansa et présente un QR de paiement à courte durée de vie. Le scanner 2D de la borne lit le jeton. Le backend résout le compte, le montant et les règles de sécurité avant autorisation.

Le QR ne doit pas contenir de secret durable ni d’information financière complète en clair.

### 4.2 Le client scanne le QR de la borne

La borne affiche un QR dynamique lié à :

- une transaction unique ;
- un terminal ;
- une voie ;
- un montant ;
- une devise ;
- une expiration courte ;
- une référence de corrélation.

Le client scanne ce QR avec l’application Mansa. L’ouverture de la barrière intervient uniquement après confirmation serveur de la transaction.

Un QR statique public ne doit pas être utilisé comme preuve suffisante d’un paiement de péage.

## 5. Carte bancaire et sans-contact

Le module carte est isolé du logiciel métier. Le TPE certifié gère les fonctions EMV/NFC et les exigences de sécurité de l’acquéreur.

Mansa reçoit un résultat de paiement et une référence externe, sans stocker de PAN complet, PIN ou donnée de piste.

La borne doit pouvoir désactiver `BANK_CARD` lorsqu’un terminal est hors ligne tout en maintenant les autres moyens disponibles.

## 6. Mobile Money

Le paiement Mobile Money est traité par un adaptateur partenaire. La disponibilité peut être activée ou désactivée par organisation, pays, voie ou période.

Une indisponibilité Mobile Money ne doit pas bloquer la voie si une autre méthode autorisée reste disponible.

## 7. Billets et pièces FCFA

Mansa ne doit pas déterminer lui-même qu’un billet ou une pièce est authentique à partir d’une image générique. Cette fonction est confiée à un validateur de monnaie industriel configuré et accepté pour la devise concernée.

Pour le Mali, le fournisseur de la borne doit confirmer explicitement la compatibilité avec la devise `XOF` et les coupures/pièces configurées avant déploiement.

Le profil terminal conserve les valeurs autorisées en unités mineures :

```text
supportedCurrencies
acceptedBillDenominationsMinor
acceptedCoinDenominationsMinor
canGiveChange
```

La liste configurée dans Mansa ne remplace pas la validation matérielle : elle indique seulement ce que la plateforme autorise à accepter.

## 8. Validation des espèces

Le validateur matériel analyse la coupure et retourne un résultat structuré. Mansa enregistre l’événement sans prétendre établir juridiquement la contrefaçon.

Résultats :

```text
ACCEPTED
REJECTED_UNKNOWN_DENOMINATION
REJECTED_SUSPECT
REJECTED_DAMAGED
REJECTED_DISABLED_DENOMINATION
REJECTED_CASHBOX_FULL
```

`REJECTED_SUSPECT` signifie que le validateur n’a pas accepté l’instrument. L’interface publique doit employer un message neutre, par exemple :

```text
Billet non accepté. Reprenez votre billet et utilisez une autre coupure ou un autre moyen de paiement.
```

La borne ne doit pas afficher « faux billet » sur la seule base de ce résultat.

## 9. Rendu de monnaie

Si `canGiveChange = false`, l’écran doit prévenir le conducteur avant insertion d’espèces et appliquer une règle de paiement compatible avec le tarif.

Si `canGiveChange = true`, la borne utilise des recycleurs de billets/pièces ou des modules dédiés. Le logiciel suit :

- montant dû ;
- montant introduit ;
- montant accepté ;
- monnaie à rendre ;
- résultat de distribution ;
- éventuel incident de rendu.

Aucune transaction ne doit être marquée terminée avec un solde incohérent.

## 10. Profil matériel minimal Mansa

Pour une borne extérieure de péage complète :

```text
écran tactile
scanner QR 2D
TPE EMV/NFC
validateur billets compatible devise cible
monnayeur compatible devise cible
recycleur si rendu de monnaie requis
imprimante reçu
interphone
contrôleur de voie
liaison barrière
capteurs véhicule
réseau principal + stratégie dégradée
coffre sécurisé
```

Le RFID UHF et la caméra ANPR peuvent être installés en amont de la borne et pilotés par le même contrôleur de voie.

## 11. Pannes et affichage

L’écran doit refléter l’état réel du service. Exemples :

- `Carte temporairement indisponible — utilisez QR Mansa ou espèces.`
- `Paiement en billets indisponible — pièces et carte disponibles.`
- `Voie fermée — utilisez la voie 2.`
- `Service en mode dégradé.`

Une panne de composant ne doit jamais être présentée comme un refus du client.

Lorsqu’un composant est défaillant, seule la fonction correspondante est retirée de la liste des moyens disponibles, sauf si la sécurité impose la fermeture complète de la voie.

## 12. Contrats techniques

Le code de référence se trouve dans :

- `mansa-platform/packages/contracts/src/access-mobility.ts` ;
- `mansa-platform/packages/contracts/src/access-mobility-api.ts`.

Objets principaux ajoutés pour les bornes :

```text
AccessTerminalProfile
AccessCashValidationEvent
AccessTerminalDisplayState
AccessServiceAvailability
```

Routes :

```text
GET  /v1/access/terminals/:terminalId/profile
GET  /v1/access/terminals/:terminalId/display-state
POST /v1/access/terminals/:terminalId/cash-validations
```

Les événements de validation cash utilisent idempotence et corrélation. Ils ne contiennent aucune image de billet par défaut.

## 13. Sécurité et exploitation

Obligations :

- aucun secret matériel dans Git ;
- configuration sensible injectée par environnement ou gestionnaire de secrets ;
- journalisation de l’état des périphériques ;
- alerte coffre plein ;
- procédure sécurisée de collecte d’espèces ;
- séparation des rôles entre maintenance, exploitation et rapprochement financier ;
- traçabilité des ouvertures manuelles ;
- rapprochement quotidien entre transactions, ledger et montants collectés ;
- surveillance du taux de rejet par validateur ;
- maintenance préventive des modules cash.

## 14. Critères d’acceptation

Le lot borne est accepté lorsque :

1. un abonné RFID valide passe sans devoir sélectionner RFID à l’écran ;
2. un non-abonné voit le montant et uniquement les méthodes réellement disponibles ;
3. la borne supporte au moins un mode QR Mansa dynamique ;
4. une borne `DUAL_HEIGHT` représente correctement les interactions voiture et camion ;
5. billets et pièces ne sont crédités qu’après acceptation du validateur matériel ;
6. une coupure rejetée ne modifie pas le montant encaissé ;
7. un validateur indisponible retire uniquement le cash concerné lorsque les autres méthodes restent sûres ;
8. une panne est clairement affichée ;
9. toute validation cash est corrélée et idempotente ;
10. aucune donnée de carte sensible ni secret matériel n’est journalisé.

## 15. À valider avant achat

Avant commande d’une borne réelle, demander au fournisseur une confirmation écrite de :

- compatibilité exacte `XOF` des validateurs billets et pièces ;
- liste des coupures/pièces reconnues ;
- capacité de mise à jour des jeux de données monnaie ;
- résistance extérieure et plage de température ;
- indice IP/IK ;
- disponibilité des pièces de rechange ;
- API/protocole de chaque périphérique ;
- double hauteur réelle pour voitures et cabines de camion ;
- certification TPE et compatibilité acquéreur ;
- options de coffre, anti-effraction et détection d’ouverture ;
- garantie et support.
