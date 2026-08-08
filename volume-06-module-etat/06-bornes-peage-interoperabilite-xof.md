# Bornes de péage Mansa — interopérabilité, double hauteur, espèces XOF et rendu de monnaie

## 1. Objet

Ce document fixe les exigences fonctionnelles et techniques des bornes de péage utilisées avec Mansa. Il ne définit pas une borne propriétaire Mansa : la plateforme doit pouvoir intégrer des équipements de plusieurs constructeurs et des installations déjà détenues par l’État, un concessionnaire ou une entreprise.

Principe impératif :

```text
Mansa -> Mansa Kiosk Gateway -> adaptateur constructeur/protocole -> périphériques de la borne
```

Aucune règle métier critique ne doit être enfermée dans le firmware d’un fabricant.

## 2. Profil recommandé pour un nouveau péage

Pour une voie destinée à recevoir voitures, utilitaires, bus et poids lourds, la borne de référence doit être extérieure et double hauteur.

Elle doit pouvoir intégrer :

- interface basse pour voitures ;
- interface haute pour bus et poids lourds ;
- écran lisible en extérieur ;
- terminal carte EMV avec puce et sans-contact/NFC ;
- scanner QR 2D ;
- affichage d’un QR dynamique ;
- Mobile Money lorsque l’organisation l’active ;
- validateur de billets ;
- validateur de pièces ;
- recycleur de billets et/ou de pièces ;
- rendu de monnaie ;
- imprimante de reçu ;
- interphone ou bouton d’assistance ;
- coffre sécurisé ;
- contrôleur local ;
- liaison vers barrière, capteurs, RFID UHF et ANPR ;
- Ethernet et options de connectivité de secours selon le site ;
- alimentation secourue lorsque nécessaire.

Le matériel réel peut avoir moins de capacités. Mansa doit détecter les capacités disponibles et n’afficher que les moyens opérationnels.

## 3. Compatibilité multi-constructeurs

Les intégrations peuvent utiliser, selon le matériel :

- API HTTP/REST locale ;
- webhook ;
- SDK constructeur ;
- Ethernet/TCP-IP ;
- USB ;
- RS-232 ;
- RS-485 ;
- MDB ;
- Pulse ;
- GPIO ;
- relais/contact sec ;
- autre protocole documenté et auditable.

Une borne propriétaire sans interface exploitable peut nécessiter une passerelle matérielle dédiée. Cette limite doit être documentée pendant l’onboarding et ne doit jamais être masquée par une promesse de compatibilité universelle.

## 4. Profil de capacités

Chaque borne publie un profil de capacités, par exemple :

```text
CARD_EMV
CARD_NFC
QR_SCANNER
QR_DISPLAY
MOBILE_MONEY
CASH_BILL_ACCEPT
CASH_COIN_ACCEPT
CASH_BILL_CHANGE
CASH_COIN_CHANGE
RECEIPT_PRINTER
INTERCOM
RFID_UHF
ANPR
BARRIER_CONTROL
VEHICLE_SENSORS
OFFLINE_MODE
```

L’interface utilisateur de la borne est dérivée de ce profil et de l’état de santé courant des périphériques.

## 5. RFID et ANPR

Le RFID UHF et l’ANPR sont des mécanismes d’identification automatique, pas des boutons de paiement à proposer par défaut sur l’écran.

Pour un véhicule abonné :

```text
arrivée -> RFID/plaque -> vérification abonnement -> décision -> barrière
```

Si aucun droit valide n’est trouvé, la borne affiche le montant dû et les moyens de paiement réellement disponibles.

## 6. QR Mansa

Deux modes sont autorisés :

- `TERMINAL_SCANS_CUSTOMER_QR` : le client présente son QR à la borne ;
- `CUSTOMER_SCANS_DYNAMIC_TERMINAL_QR` : la borne génère un QR de transaction à durée courte que le client scanne dans Mansa.

Le QR dynamique doit être lié au terminal, à la transaction, au montant, à la devise, à une date d’expiration et à une référence non réutilisable. Aucun secret long terme ne doit être encodé dans le QR.

## 7. Exigence XOF / FCFA BCEAO

Une borne ne doit jamais être déclarée compatible FCFA uniquement parce qu’elle accepte des espèces dans une autre devise.

Avant achat ou recette au Mali, le fournisseur doit confirmer par écrit et démontrer :

- les coupures XOF acceptées ;
- les pièces XOF acceptées ;
- l’authentification des billets ;
- le rejet des billets suspects, inconnus ou non autorisés ;
- l’authentification et le rejet des pièces non reconnues ;
- le firmware ou dataset monétaire XOF compatible ;
- les coupures utilisées pour le rendu de monnaie ;
- la capacité des recycleurs et hoppers ;
- la procédure de mise à jour lors d’une évolution des billets ou pièces ;
- les taux de reconnaissance mesurés en recette ;
- la tenue aux conditions du site : chaleur, poussière, humidité, vibrations et usage intensif.

Le validateur monétique spécialisé prend la décision d’acceptation de l’espèce. Mansa reçoit une valeur normalisée, une devise et un statut. Mansa ne doit pas implémenter une détection artisanale de faux billets via caméra générale.

## 8. Rendu de monnaie

La borne peut supporter plusieurs politiques :

```text
EXACT_CHANGE_REQUIRED
CHANGE_COINS_ONLY
CHANGE_BILLS_ONLY
CHANGE_MIXED
EXACT_PAYMENT_ONLY_FALLBACK
DISABLE_CASH_IF_CHANGE_UNAVAILABLE
```

Avant d’accepter un paiement espèces, le contrôleur local doit vérifier que la politique peut être satisfaite avec le stock courant des recycleurs.

Le portail d’exploitation suit au minimum :

- montant disponible par cassette/recycleur ;
- seuil bas ;
- seuil haut ;
- cassette pleine ;
- cassette absente ;
- périphérique bloqué ;
- dernière collecte ;
- dernier remplissage ;
- écarts de comptage ;
- alertes de maintenance.

## 9. Dégradation partielle

Une panne d’un périphérique ne doit pas désactiver inutilement toute la voie.

Exemples :

- monnayeur hors service : retirer espèces, garder carte/QR si disponibles ;
- rendu de monnaie indisponible : passer en paiement exact seulement si la politique l’autorise ;
- imprimante hors service : proposer reçu numérique si autorisé ;
- RFID indisponible : basculer vers paiement ponctuel ou autre contrôle ;
- ANPR indisponible : appliquer la politique de secours configurée ;
- réseau externe indisponible : utiliser uniquement les opérations explicitement autorisées hors ligne.

Chaque bascule doit être auditée.

## 10. Onboarding d’une borne existante

Mansa collecte :

- constructeur ;
- modèle ;
- numéro logique d’équipement ;
- version firmware ;
- OS ou contrôleur embarqué ;
- protocoles disponibles ;
- SDK/API disponibles ;
- interfaces physiques ;
- périphériques installés ;
- devises configurées ;
- capacités de rendu de monnaie ;
- connectivité ;
- état de maintenance ;
- documentation technique associée.

Le résultat du diagnostic est classé :

```text
NATIVE_INTEGRATION
EXISTING_SOFTWARE_ADAPTER
HARDWARE_GATEWAY_REQUIRED
PERIPHERAL_REPLACEMENT_REQUIRED
UNSUPPORTED
```

## 11. Abstractions techniques

Le code plateforme doit converger vers des interfaces indépendantes des fabricants :

```text
KioskProvider
PaymentTerminalProvider
CashBillValidatorProvider
CashCoinValidatorProvider
CashRecyclerProvider
QrScannerProvider
ReceiptPrinterProvider
IntercomProvider
ANPRProvider
RFIDReaderProvider
BarrierProvider
LaneControllerProvider
VehicleSensorProvider
```

Chaque adaptateur expose :

- ses capacités ;
- son état de santé ;
- sa version ;
- les erreurs normalisées ;
- les événements entrants ;
- les commandes supportées.

## 12. Sécurité

Obligatoire :

- aucun secret dans Git ;
- authentification mutuelle ou mécanisme équivalent lorsque supporté ;
- chiffrement des échanges réseau ;
- liste blanche des commandes matérielles ;
- signature/corrélation des événements sensibles ;
- contrôle d’accès au mode maintenance ;
- audit des ouvertures manuelles ;
- chiffrement ou tokenisation des données de paiement selon périmètre ;
- aucun PAN complet dans les logs ;
- aucun code PIN traité par l’application générale Mansa en dehors du composant de paiement certifié.

## 13. Critères d’acceptation

Le lot borne est recevable lorsque :

1. une borne double hauteur peut être modélisée sans logique spécifique au constructeur ;
2. les capacités matérielles sont déclarées et interrogables ;
3. l’écran masque automatiquement un moyen indisponible ;
4. un paiement QR Mansa peut utiliser les deux modes prévus ;
5. RFID/ANPR restent séparés des boutons de paiement ;
6. les espèces XOF ne sont activées qu’après validation explicite du matériel monétique ;
7. le rendu de monnaie est piloté par politique et stock réel ;
8. une panne partielle conserve les moyens sains lorsque c’est sûr ;
9. les événements et décisions sont corrélés et auditables ;
10. l’intégration d’un autre constructeur ne nécessite pas de réécrire le moteur métier.

## 14. Éléments à valider avec les fournisseurs

Avant achat réel restent obligatoirement à confirmer :

- compatibilité officielle XOF/BCEAO ;
- liste précise des billets et pièces supportés ;
- certifications monétiques et de paiement applicables ;
- prix selon configuration ;
- disponibilité des pièces détachées ;
- maintenance locale ou régionale ;
- SDK/API et droits de redistribution ;
- résistance environnementale ;
- durée de support firmware ;
- tests de recette usine puis site.
