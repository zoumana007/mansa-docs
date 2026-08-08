# Bornes, incidents et continuité de service

## 1. Objet

Ce document complète le moteur `Accès & Mobility` pour les bornes de péage, parking, transport, campus, flotte et autres points de passage. Il définit les états de service, les pannes partielles, les moyens de paiement affichés, les règles RFID + plaque, le fonctionnement dégradé et le traitement des abonnements en cas de suspension ou de résiliation.

La borne ne doit jamais supposer qu’un seul moyen d’identification ou de paiement existe. Les capacités disponibles dépendent de la configuration du site, de la voie, du terminal et de l’état de santé des équipements.

## 2. Moyens pris en charge

Le socle partagé distingue notamment :

- carte bancaire ;
- Mobile Money ;
- billets ;
- pièces ;
- QR Mansa ;
- solde prépayé ;
- compte postpayé ;
- abonnement ;
- RFID UHF pour les véhicules abonnés ;
- NFC pour les cartes et badges ;
- plaque ANPR comme identifiant ou facteur de contrôle complémentaire.

Les méthodes réellement proposées à l’usager sont calculées à partir du produit, de la politique de l’organisation et de l’état des équipements.

## 3. RFID + plaque

Pour un véhicule abonné, la politique recommandée est une vérification croisée du tag RFID et de la plaque lorsque l’ANPR est disponible.

Politiques supportées par contrat :

```text
CREDENTIAL_ONLY
PLATE_ONLY
CREDENTIAL_AND_PLATE_REQUIRED
CREDENTIAL_AND_PLATE_PREFERRED
CREDENTIAL_VALID_PLATE_UNREADABLE_ALLOW_WITH_RULES
CREDENTIAL_VALID_PLATE_MISMATCH_DENY
MANUAL_REVIEW
```

Exemples :

- RFID valide + plaque attendue : passage autorisable ;
- RFID valide + plaque différente avec confiance suffisante : refus ou contrôle manuel selon politique ;
- RFID valide + plaque illisible : application de la politique dégradée ;
- RFID suspendu ou révoqué : refus, même si la plaque correspond.

## 4. États du service

Un service, un site ou une voie peut être :

```text
ACTIVE
SUSPENDED
MAINTENANCE
DEGRADED
CLOSED
DISABLED
```

`DEGRADED` signifie que le service continue avec un sous-ensemble sûr de fonctionnalités. Exemple : la caméra ANPR est indisponible mais le RFID et le paiement manuel restent opérationnels.

La transition d’état doit être horodatée, justifiée et auditée.

## 5. Santé des équipements

Le contrat partagé suit au minimum les composants suivants :

```text
RFID_READER
ANPR_CAMERA
CARD_TERMINAL
MOBILE_MONEY_GATEWAY
CASH_ACCEPTOR
COIN_ACCEPTOR
QR_SCANNER
BARRIER
LANE_CONTROLLER
VEHICLE_SENSOR
RECEIPT_PRINTER
DISPLAY
NETWORK
```

Chaque composant peut être `ONLINE`, `DEGRADED`, `OFFLINE`, `MAINTENANCE` ou `DISABLED`.

Une panne d’un composant ne doit pas provoquer une fermeture totale si un mode sûr reste disponible.

## 6. Affichage de la borne

L’écran doit toujours présenter un état cohérent avec le service réel. Les messages sont localisables et référencés par clé plutôt que codés en dur dans les terminaux.

Exemples d’affichage :

```text
Service temporairement indisponible — utilisez la voie 2.
Télépéage indisponible — veuillez utiliser le paiement à la borne.
Paiement par carte indisponible — choisissez un autre moyen.
Connexion momentanément indisponible — fonctionnement en mode dégradé.
Voie fermée pour maintenance.
```

Une panne technique ne doit jamais être affichée comme un refus de paiement du client.

## 7. Borne non abonné

Lorsqu’aucun droit d’abonnement valide n’est détecté, la borne peut proposer un paiement à l’usage selon la configuration :

```text
Carte bancaire
Mobile Money
Billets
Pièces
QR Mansa
```

La borne affiche le tarif avant confirmation. Après succès, elle affiche le montant, le moyen de paiement, l’heure, la voie, le statut et la disponibilité du reçu.

## 8. Pannes partielles et bascule

Exemples de comportements :

- terminal carte hors ligne : masquer la carte et conserver les autres moyens ;
- Mobile Money indisponible : masquer ce canal sans bloquer la voie ;
- lecteur RFID hors ligne : proposer le passage manuel ou le paiement à l’usage ;
- ANPR indisponible : appliquer la politique dégradée autorisée ;
- imprimante en panne : paiement possible si le reçu numérique est disponible et si la politique l’autorise ;
- barrière ou capteur de sécurité en panne : fermer la voie si la sécurité n’est plus garantie ;
- réseau indisponible : utiliser uniquement les droits et limites autorisés par la politique offline.

Toutes les bascules sont tracées.

## 9. API de disponibilité

Le contrat technique de référence est défini dans :

- `mansa-platform/packages/contracts/src/access-mobility.ts` ;
- `mansa-platform/packages/contracts/src/access-mobility-api.ts`.

Routes de référence :

```text
GET /v1/access/locations/:locationId/availability
PUT /v1/access/locations/:locationId/availability
GET /v1/access/terminals/:terminalId/display-state
```

La mise à jour d’un état de service exige un motif, une clé d’idempotence et un identifiant de corrélation.

## 10. Abonnements et résiliation

Un abonnement peut être :

```text
DRAFT
ACTIVE
SUSPENDED
EXPIRED
CANCELLED
TERMINATED
```

La suspension ou la résiliation ne supprime jamais l’historique de paiement, de passage, de véhicule ou d’affectation du credential.

La politique financière doit être configurable :

```text
NON_REFUNDABLE
PRORATA_REFUND
CREDIT
EXTEND_VALIDITY
MANUAL_DECISION
```

Un organisme public peut choisir `NON_REFUNDABLE` si son cadre juridique et ses conditions contractuelles le permettent. Cette politique ne doit pas être imposée globalement dans le code.

## 11. Suspension du service et compensation

Quand le service est suspendu indépendamment de l’usager, la politique peut être :

```text
SUBSCRIPTION_CLOCK_CONTINUES
PAUSE_AND_EXTEND
COMPENSATE_WITH_CREDIT
MANUAL_COMPENSATION
NO_COMPENSATION
```

La politique applicable doit être versionnée et datée. Un changement ne doit pas modifier silencieusement les conditions d’un abonnement déjà souscrit.

## 12. Données d’incident

Un incident doit pouvoir enregistrer :

- organisation ;
- site ;
- voie ;
- terminal ;
- équipement concerné ;
- état avant et après ;
- heure de début et de fin ;
- cause ou code de panne ;
- message présenté à l’usager ;
- moyens de paiement restés disponibles ;
- voie de secours éventuelle ;
- nombre de passages affectés ;
- décisions manuelles ;
- politique de compensation ;
- auteur de la modification ;
- identifiant de corrélation.

## 13. Critères d’acceptation

Le lot est acceptable lorsque :

1. une panne partielle ne masque que les capacités réellement indisponibles ;
2. l’écran de la borne reflète l’état réel du service ;
3. un véhicule peut être contrôlé par RFID + plaque selon une politique configurable ;
4. un véhicule non abonné peut recevoir les moyens de paiement autorisés ;
5. une voie dangereuse peut être fermée automatiquement ou manuellement ;
6. les transitions de service et incidents sont auditables ;
7. une suspension ne supprime aucun historique ;
8. la politique de remboursement ou compensation est explicite ;
9. les règles de disponibilité sont exposées via les contrats partagés ;
10. aucun secret matériel ou fournisseur n’est stocké dans le dépôt.

## 14. Hors production tant que non validé

Restent à valider avec les partenaires et autorités avant production :

- messages réglementaires obligatoires sur les bornes ;
- règles de remboursement des abonnements publics ;
- procédures d’ouverture manuelle ;
- comportement exact en cas de panne de réseau ;
- modèles de caméras ANPR et taux de confiance ;
- protocoles des lecteurs RFID, barrières, accepteurs de billets et monnayeurs ;
- homologations électriques, radio et paiement ;
- obligations de conservation des images et plaques.
