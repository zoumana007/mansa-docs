# Interopérabilité des bornes et Mansa Kiosk Gateway

## 1. Objectif

Mansa ne doit dépendre d'aucune marque, d'aucun fabricant ni d'un modèle unique de borne. Une administration, un exploitant autoroutier, un parking, une entreprise ou un partenaire doit pouvoir raccorder une borne existante ou future dès lors qu'une interface technique exploitable est disponible.

La borne double hauteur compatible XOF constitue le profil recommandé pour les nouvelles voies mixtes voitures/poids lourds, mais elle n'est pas une dépendance de la plateforme.

Architecture cible :

```text
Mansa Cloud / services métier
        |
        v
Mansa Kiosk Gateway
        |
        +-- adaptateur constructeur A
        +-- adaptateur constructeur B
        +-- adaptateur protocole standard
        |
        v
Contrôleur de voie / périphériques de borne
```

## 2. Responsabilités du Kiosk Gateway

Le `Mansa Kiosk Gateway` constitue la couche d'abstraction entre les contrats métier Mansa et les protocoles matériels. Il doit :

- découvrir ou charger les capacités déclarées d'une borne ;
- normaliser les états de santé ;
- normaliser les événements matériels ;
- traduire les commandes Mansa vers le protocole du constructeur ;
- isoler les SDK propriétaires du coeur métier ;
- gérer timeouts, reprises, idempotence et corrélation ;
- permettre un mode local dégradé lorsque la politique l'autorise ;
- ne jamais embarquer de secret en dur dans Git ;
- journaliser la version de l'adaptateur et du firmware ;
- exposer des diagnostics exploitables par le portail d'administration.

Le Gateway n'est pas le ledger et ne décide pas seul qu'un paiement financier est définitivement réglé. Il relaie les résultats des périphériques et applique uniquement les règles locales explicitement autorisées.

## 3. Protocoles et interfaces à prévoir

Un adaptateur peut utiliser, selon la borne :

```text
HTTP/REST local
webhook
SDK constructeur
Ethernet / TCP-IP
USB
RS-232
RS-485
MDB
Pulse
GPIO
relais / contact sec
Wiegand
ONVIF lorsque pertinent
```

Cette liste n'est pas une promesse de compatibilité automatique. Chaque protocole doit être pris en charge par un adaptateur testé. Une borne ancienne ou totalement propriétaire peut nécessiter une passerelle matérielle supplémentaire.

## 4. Profil de capacités

Chaque terminal doit publier un profil de capacités et Mansa ne doit afficher que les fonctions réellement présentes et opérationnelles.

Capacités logiques minimales à représenter :

```text
BANK_CARD
NFC
QR_SCANNER
QR_DISPLAY
MOBILE_MONEY
CASH_BILL_ACCEPT
CASH_COIN_ACCEPT
CASH_BILL_CHANGE
CASH_COIN_CHANGE
RECEIPT_PRINT
INTERCOM
RFID_UHF
ANPR
BARRIER_CONTROL
VEHICLE_SENSORS
OFFLINE_MODE
```

Le contrat actuel `AccessTerminalProfile` expose déjà les éléments métier suivants :

- profil de hauteur ;
- méthodes de paiement ;
- modes QR ;
- devises supportées ;
- coupures de billets autorisées ;
- pièces autorisées ;
- capacité de rendre la monnaie ;
- présence d'une imprimante ;
- présence d'un interphone.

La couche Gateway détaillera ensuite les capacités physiques par périphérique sans casser le contrat métier.

## 5. Double hauteur

Pour une voie mixte, le profil recommandé reste :

```text
DUAL_HEIGHT
```

Les interfaces basse et haute peuvent disposer de périphériques dupliqués ou partager certains modules internes. Dans tous les cas :

- elles représentent une seule transaction de voie ;
- elles affichent le même montant dû ;
- elles doivent empêcher deux paiements concurrents pour le même passage ;
- le premier canal de paiement verrouillé devient propriétaire temporaire de la transaction ;
- un abandon ou timeout libère proprement ce verrou.

## 6. XOF / FCFA BCEAO

La compatibilité XOF doit être prouvée par le fournisseur du validateur ou du recycleur, puis validée en recette. Mansa ne doit jamais déduire la compatibilité XOF du simple fait qu'un périphérique accepte une autre monnaie.

Le dossier fournisseur doit préciser au minimum :

- devise `XOF` ;
- séries de billets reconnues ;
- coupures acceptées ;
- pièces acceptées ;
- méthode de mise à jour du dataset monétaire ;
- capacités de rejet ;
- capacité de recyclage ;
- coupures utilisables pour le rendu ;
- capacité des cassettes/hoppers ;
- comportement coffre plein ;
- comportement monnaie insuffisante ;
- taux de reconnaissance déclaré ;
- maintenance et nettoyage ;
- plage de température et conditions environnementales.

Mansa enregistre un résultat tel que `REJECTED_SUSPECT` mais ne doit pas afficher « faux billet » ni conclure juridiquement à une contrefaçon sur cette seule information.

## 7. Rendu de monnaie

Le rendu de monnaie doit être une capacité réelle et surveillée, pas un simple booléen statique de configuration.

Le Gateway doit remonter :

- disponibilité du recycleur ;
- inventaire logique par coupure ;
- seuil bas ;
- seuil haut ;
- cassette pleine ;
- cassette absente ;
- bourrage ;
- distribution partielle ;
- distribution réussie ;
- montant restant à rendre.

Stratégies métier configurables :

```text
EXACT_CHANGE_REQUIRED
CHANGE_BILLS_ONLY
CHANGE_COINS_ONLY
CHANGE_MIXED
EXACT_PAYMENT_ONLY_WHEN_LOW_FLOAT
DISABLE_CASH_WHEN_CHANGE_UNAVAILABLE
```

Avant de commencer une transaction cash, le système doit vérifier qu'il peut respecter la stratégie en vigueur. Une transaction ne doit pas être clôturée si le rendu attendu et le rendu réellement distribué divergent sans traitement explicite d'incident.

## 8. Abstraction des périphériques

Interfaces de programmation recommandées :

```text
KioskProvider
LaneControllerProvider
PaymentTerminalProvider
BillValidatorProvider
CoinValidatorProvider
CashRecyclerProvider
QrScannerProvider
DisplayProvider
ReceiptPrinterProvider
IntercomProvider
RFIDReaderProvider
ANPRProvider
BarrierProvider
VehicleSensorProvider
```

Chaque adaptateur publie :

- `provider` ;
- `model` ;
- `firmwareVersion` si disponible ;
- `adapterVersion` ;
- protocoles actifs ;
- capacités ;
- état ;
- dernière communication ;
- code erreur normalisé ;
- diagnostics constructeur conservés séparément.

## 9. Onboarding d'une borne existante

Flux recommandé :

1. identifier fabricant et modèle ;
2. relever firmware et contrôleur ;
3. inventorier les périphériques ;
4. récupérer la documentation technique ;
5. identifier API, SDK ou protocole ;
6. vérifier qu'aucun secret ne sera stocké dans Git ;
7. choisir ou créer l'adaptateur ;
8. exécuter les tests de lecture/commande en environnement isolé ;
9. déclarer les capacités réellement validées ;
10. exécuter une recette de voie ;
11. activer progressivement ;
12. conserver le rapport de compatibilité.

Résultats possibles :

```text
NATIVE_SUPPORTED
SUPPORTED_WITH_SOFTWARE_ADAPTER
SUPPORTED_WITH_HARDWARE_GATEWAY
PARTIALLY_SUPPORTED
UNSUPPORTED
```

Une incompatibilité documentée est préférable à une fausse promesse de compatibilité universelle.

## 10. Tests de conformité d'un adaptateur

Tout nouvel adaptateur doit couvrir :

- connexion/reconnexion ;
- timeout ;
- événement dupliqué ;
- redémarrage du Gateway ;
- perte réseau cloud ;
- perte réseau périphérique ;
- périphérique indisponible ;
- retour à l'état nominal ;
- commandes idempotentes lorsque nécessaire ;
- corrélation bout en bout ;
- absence de secrets dans les logs ;
- sécurité de la barrière ;
- cohérence du paiement avec le ledger ;
- affichage des moyens réellement disponibles.

Pour le cash :

- chaque coupure autorisée ;
- coupure désactivée ;
- billet rejeté ;
- pièce rejetée ;
- coffre plein ;
- bourrage ;
- rendu exact ;
- rendu mixte ;
- monnaie insuffisante ;
- incident de distribution ;
- reprise après maintenance.

## 11. Sécurité

Les SDK propriétaires, clés API, mots de passe de borne, certificats privés et identifiants de maintenance ne doivent jamais être commités. La configuration sensible est injectée par gestionnaire de secrets ou mécanisme sécurisé équivalent.

Le Gateway doit appliquer le principe du moindre privilège. Un adaptateur chargé de lire un validateur de billets ne doit pas obtenir par défaut des droits d'administration cloud ou de modification du ledger.

Les commandes dangereuses, notamment ouverture manuelle de barrière, mode maintenance ou désactivation d'un périphérique de sécurité, doivent être auditées et protégées par des autorisations explicites.

## 12. Cohérence avec le contrat Mansa actuel

Cette spécification complète sans remplacer :

- `AccessTerminalProfile` ;
- `AccessCashValidationEvent` ;
- `AccessTerminalDisplayState` ;
- `AccessServiceAvailability` ;
- `AccessEquipmentHealth`.

Les valeurs déjà prévues pour `CASH_RECYCLER`, `COIN_RECYCLER`, `QR_SCANNER`, `INTERCOM`, `DUAL_HEIGHT` et les deux modes QR restent la source de vérité côté contrat partagé.

Les détails protocolaires et inventaires de recycler appartiennent à la couche d'intégration matérielle et ne doivent pas polluer les objets financiers génériques.

## 13. Critères d'acceptation du lot Gateway

Le lot est considéré prêt lorsque :

1. aucun code métier ne dépend du nom d'un fabricant ;
2. au moins un adaptateur de référence peut être substitué par un simulateur sans modifier le moteur métier ;
3. les capacités pilotent réellement l'affichage de la borne ;
4. un périphérique en panne peut être retiré sans fermer toute la voie si la sécurité le permet ;
5. les événements matériels sont corrélés et dédupliqués ;
6. le mode dégradé respecte des limites explicites ;
7. le cash XOF n'est activable qu'après validation du profil monétaire ;
8. la capacité de rendu est contrôlée avant acceptation cash selon la politique ;
9. les secrets ne sont ni dans Git ni dans les logs ;
10. une recette d'adaptateur est archivée pour chaque modèle déployé.
