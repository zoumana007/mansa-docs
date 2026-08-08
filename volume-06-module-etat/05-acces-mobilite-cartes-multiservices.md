# Accès, mobilité et cartes multiservices Mansa

## 1. Objet

Ce document définit le moteur transversal d’identification, d’autorisation, d’abonnement et de traçabilité utilisable par le secteur public comme par les organisations privées. Le péage routier est un cas d’usage parmi d’autres : parking, transport public, campus, accès d’entreprise, restauration, station-service et gestion de flotte.

Le moteur doit rester multi-tenant et indépendant d’un fabricant de lecteur, d’une marque de carte ou d’un opérateur unique.

## 2. Principe commun

Le flux générique est :

```text
identifiant -> résolution du sujet -> contrôle du droit -> décision -> paiement éventuel -> action physique -> journal d’audit
```

Un identifiant peut être :

- carte NFC ;
- tag RFID UHF ;
- QR code ;
- plaque d’immatriculation lorsqu’un système de lecture est autorisé ;
- jeton d’appareil pour certains usages fermés.

Aucun identifiant physique ne doit constituer à lui seul une preuve suffisante pour une opération financière sensible. Le backend vérifie toujours son statut, son organisation, sa période de validité et les règles associées.

## 3. Choix technologiques

### NFC

À privilégier pour les cartes et badges présentés volontairement à courte distance :

- cartes étudiantes ;
- badges employés ;
- validation transport ;
- restauration ;
- contrôle d’accès ;
- événements.

### RFID UHF

À privilégier lorsque l’identification doit se faire à plusieurs mètres, notamment pour :

- télépéage ;
- parking véhicule ;
- accès de flotte ;
- stations-service automatisées.

La technologie UHF ne doit pas être utilisée comme unique mécanisme d’autorisation : un tag copié, désactivé ou signalé doit être refusé par les règles serveur.

## 4. Cas d’usage obligatoires

Le moteur doit au minimum supporter :

- `TOLL` : péage et télépéage ;
- `PARKING` : accès et facturation de parking ;
- `PUBLIC_TRANSPORT` : titres, abonnements et validations ;
- `CAMPUS` : carte étudiante et services universitaires ;
- `EMPLOYEE_ACCESS` : badge salarié et zones autorisées ;
- `FUEL_FLEET` : station-service et flotte entreprise ;
- `CANTEEN` : restauration subventionnée ou privée ;
- `EVENT` : contrôle d’entrée temporaire.

Les nouveaux secteurs doivent être ajoutables sans modifier le moteur de décision fondamental.

## 5. Carte étudiante multiservice

Une carte étudiante peut regrouper, selon les droits accordés par l’établissement :

- identité étudiante de référence ;
- période académique ;
- accès campus ;
- bibliothèque ;
- restauration ;
- logement ;
- transport ;
- bourses ou aides visibles dans l’application ;
- paiements autorisés ;
- QR de secours ;
- identifiant NFC.

Les données sensibles, secrets d’authentification et numéros financiers complets ne sont jamais stockés en clair sur la carte.

## 6. Entreprises et employés

Une entreprise peut créer des droits par salarié, équipe, véhicule ou site :

- badge d’accès ;
- horaires ;
- zones ;
- parking ;
- restauration ;
- transport interne ;
- dépenses de flotte ;
- plafonds et règles d’utilisation.

Les droits sont configurables depuis le portail de l’organisation sans redéploiement du code lorsque la règle appartient au catalogue supporté.

## 7. Station-service et flotte

Le flux cible est :

1. identification du véhicule ou du conducteur ;
2. chargement de l’organisation et du contrat de flotte ;
3. vérification du statut du badge ou tag ;
4. contrôle de la station, du carburant, du véhicule, de l’horaire et du plafond ;
5. autorisation ou refus ;
6. création du paiement ou de la créance selon le contrat ;
7. rapprochement avec la vente de carburant ;
8. émission d’un justificatif ;
9. journalisation.

Exemples de règles :

- diesel uniquement ;
- maximum 50 000 FCFA par semaine ;
- stations autorisées limitées ;
- véhicule et conducteur associés ;
- blocage immédiat d’un badge perdu ;
- double validation au-delà d’un seuil.

## 8. Transport public

Le moteur doit permettre :

- titre unitaire ;
- abonnement journalier, hebdomadaire, mensuel ou scolaire ;
- tarif étudiant ;
- tarif salarié ou conventionné ;
- validation à la montée ou au passage d’un portique ;
- contrôles hors ligne limités ;
- synchronisation dès retour du réseau ;
- lutte contre la double utilisation anormale ;
- règles par ligne, zone, opérateur ou période.

Une validation offline ne doit jamais inventer un paiement confirmé. Elle peut seulement consommer un droit préchargé ou enregistrer une preuve locale signée selon une politique configurée.

## 9. Télépéage

Pour un véhicule, le tag RFID UHF est une référence de détection et non un portefeuille autonome.

Flux :

```text
lecture UHF -> credential -> véhicule -> entitlement -> tarif -> paiement/compte -> décision -> barrière ou passage -> audit
```

Le système doit supporter :

- barrière classique ;
- voie réservée ;
- modèle free-flow futur ;
- abonnement ;
- post-paiement contractuel ;
- prépaiement ;
- liste de blocage ;
- mode dégradé strictement borné.

## 10. Modèle métier partagé

Le contrat technique de référence est défini dans :

- `mansa-platform/packages/contracts/src/access-mobility.ts` ;
- `mansa-platform/packages/contracts/src/access-mobility-api.ts`.

Concepts minimaux :

```text
AccessCredential
AccessEntitlement
AccessRequest
AccessDecision
RecordAccessUsageCommand
```

Un `AccessCredential` représente le moyen d’identification. Un `AccessEntitlement` représente le droit. Les deux doivent rester séparés : remplacer une carte ne doit pas obliger à recréer le contrat métier du bénéficiaire.

## 11. Décision

Une décision est :

```text
ALLOW
DENY
REVIEW
```

Elle doit fournir un motif structuré, par exemple :

- droit valide ;
- identifiant inconnu ;
- identifiant suspendu ;
- droit absent ;
- droit expiré ;
- quota dépassé ;
- plafond financier dépassé ;
- lieu interdit ;
- produit interdit ;
- contrôle manuel requis.

## 12. Sécurité

Principes obligatoires :

- isolation stricte des organisations ;
- identifiants publics non secrets ;
- secrets techniques jamais stockés sur les tags passifs ;
- révocation immédiate ;
- rotation/remplacement d’un credential sans perte d’historique ;
- horodatage et corrélation de chaque décision ;
- idempotence des opérations financières ;
- journal d’audit ;
- chiffrement des données sensibles côté serveur ;
- limitation des informations envoyées aux lecteurs terrain ;
- politique offline par organisation et cas d’usage.

## 13. Matériel

Mansa ne doit pas dépendre d’une marque unique. Les intégrations matérielles passent par des adaptateurs :

```text
NfcReaderAdapter
UhfReaderAdapter
BarrierControllerAdapter
QrScannerAdapter
PlateRecognitionAdapter
FuelPumpAdapter
```

Chaque adaptateur expose des capacités et un état de santé. Les règles métier restent dans Mansa et non dans le firmware du périphérique.

## 14. Administration

Une organisation autorisée doit pouvoir gérer :

- types de credentials ;
- bénéficiaires ;
- véhicules ;
- abonnements ;
- plafonds ;
- lieux ;
- produits ;
- horaires ;
- listes de blocage ;
- terminaux et lecteurs ;
- politiques offline ;
- règles tarifaires ;
- rapports d’utilisation.

Les modifications sensibles sont versionnées et auditées.

## 15. Critères d’acceptation

Le lot est considéré fonctionnel lorsque :

1. une organisation peut enregistrer un credential sans exposer de secret ;
2. un droit peut être attaché à un sujet indépendamment du credential physique ;
3. une requête d’accès produit une décision déterministe et corrélée ;
4. une carte révoquée est refusée ;
5. un droit expiré est refusé ;
6. une limite d’usage ou de montant peut provoquer un refus ;
7. le même moteur couvre au moins un scénario véhicule et un scénario personne ;
8. les événements sont auditables ;
9. les opérations financières sont idempotentes ;
10. le matériel reste remplaçable via adaptateur.

## 16. Hors périmètre immédiat

Restent à valider avec les partenaires avant production :

- fréquences radio et homologations locales ;
- fournisseurs de tags et lecteurs ;
- certification des cartes sécurisées ;
- intégration aux contrôleurs de barrières ;
- intégration aux pompes de carburant ;
- systèmes de transport existants ;
- règles juridiques liées à la lecture automatique de plaques.
