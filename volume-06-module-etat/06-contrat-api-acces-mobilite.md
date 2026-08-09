# Contrat API — Accès, mobilité et bornes multiservices

## 1. Objet

Ce document formalise le contrat HTTP partagé du domaine accès et mobilité de Mansa. Il complète la spécification fonctionnelle `05-acces-mobilite-cartes-multiservices.md` et s’aligne sur les contrats TypeScript publiés par `@mansa/contracts/access-mobility` et `@mansa/contracts/access-mobility-api` dans `mansa-platform`.

Le domaine doit rester indépendant du fabricant des lecteurs, caméras, barrières, bornes, monnayeurs, recycleurs et TPE. Les API expriment des intentions, décisions, états et événements métier. Les SDK constructeurs restent confinés dans des adaptateurs d’intégration.

## 2. Préfixe et routes de référence

Toutes les routes sont versionnées sous `/v1/access`.

| Opération | Méthode | Route |
|---|---:|---|
| Créer un moyen d’identification | POST | `/v1/access/credentials` |
| Lire un moyen d’identification | GET | `/v1/access/credentials/:credentialId` |
| Lister les moyens d’identification | GET | `/v1/access/credentials` |
| Créer un droit métier | POST | `/v1/access/entitlements` |
| Lire un droit métier | GET | `/v1/access/entitlements/:entitlementId` |
| Lister les droits métier | GET | `/v1/access/entitlements` |
| Évaluer un passage ou un accès | POST | `/v1/access/evaluate` |
| Enregistrer un usage | POST | `/v1/access/usages` |
| Lire la disponibilité d’un site | GET | `/v1/access/locations/:locationId/availability` |
| Mettre à jour la disponibilité | PUT | `/v1/access/locations/:locationId/availability` |
| Lire le profil d’une borne | GET | `/v1/access/terminals/:terminalId/profile` |
| Lire l’état d’affichage d’une borne | GET | `/v1/access/terminals/:terminalId/display-state` |
| Enregistrer une validation espèces | POST | `/v1/access/terminals/:terminalId/cash-validations` |

## 3. Moyens d’identification et droits

Un `AccessCredential` représente uniquement le moyen utilisé pour reconnaître un sujet : carte NFC, tag RFID UHF, QR, plaque ou jeton d’appareil. Il ne porte pas à lui seul le droit d’accéder.

Un `AccessEntitlement` représente le droit métier : abonnement péage, accès parking, titre de transport, badge campus, flotte carburant, restauration ou autre service. Le découplage est obligatoire afin qu’un même véhicule, usager ou membre d’organisation puisse utiliser plusieurs technologies d’identification sans dupliquer les règles métier.

La plateforme doit conserver au minimum : organisation propriétaire, sujet, statut, période de validité, cas d’usage, lieux autorisés, produits autorisés, limites d’usage et limites financières lorsque configurées.

## 4. Idempotence et corrélation

Les commandes de création et les mutations sensibles incluent une `idempotencyKey` et une `correlationId`.

L’idempotence doit empêcher la duplication d’un credential, entitlement, changement de disponibilité ou événement de validation espèces lorsque la même commande est rejouée après timeout, retry réseau ou reprise hors ligne.

La corrélation doit permettre de relier :

```text
lecture RFID / NFC / QR / ANPR
→ demande d’évaluation
→ décision
→ paiement éventuel
→ ouverture de barrière
→ passage physique
→ enregistrement d’usage
→ audit
```

## 5. Évaluation d’accès

`POST /v1/access/evaluate` reçoit un `AccessRequest` et retourne un `AccessDecision`.

La demande peut inclure :

- organisation ;
- cas d’usage ;
- type et référence du credential principal ;
- credential secondaire ;
- plaque observée et confiance ANPR ;
- politique de rapprochement ;
- site, borne et produit ;
- moyen de paiement ;
- montant demandé ;
- date d’observation ;
- identifiant de corrélation.

La décision est strictement l’une de :

```text
ALLOW
DENY
REVIEW
```

Elle doit conserver un motif explicite, notamment credential inconnu/inactif, droit absent/inactif, fenêtre temporelle invalide, limite dépassée, lieu ou produit interdit, plaque incohérente, service suspendu, équipement défaillant ou politique offline refusée.

## 6. Péage RFID + ANPR

Pour une voie télépéage, Mansa doit pouvoir appliquer plusieurs politiques configurables : credential seul, plaque seule, credential + plaque obligatoires, rapprochement préféré ou revue manuelle.

Le cas recommandé pour une voie fortement sécurisée est :

```text
tag RFID valide
+ plaque lue
+ correspondance véhicule
+ entitlement actif
+ service disponible
= ALLOW
```

Une plaque illisible ne doit pas être assimilée automatiquement à une plaque différente. Le contrat distingue `PLATE_UNREADABLE` et `PLATE_MISMATCH` afin que la politique de fallback reste explicite.

## 7. Profil matériel de borne

`GET /v1/access/terminals/:terminalId/profile` expose les capacités logiques de la borne, pas un SDK constructeur.

Le profil couvre :

- hauteur `SINGLE_LOW`, `SINGLE_HIGH` ou `DUAL_HEIGHT` ;
- moyens de paiement activés ;
- modes QR ;
- devises ;
- coupures billets et pièces acceptées ;
- capacité de rendu de monnaie ;
- imprimante reçu ;
- interphone.

Une borne double hauteur doit permettre une façade basse pour voitures/utilitaires et une façade haute pour poids lourds/bus, sans dupliquer les règles métier côté backend.

## 8. Moyens de paiement configurables

Les moyens prévus par le contrat partagé sont :

```text
BANK_CARD
MOBILE_MONEY
CASH_BILLS
CASH_COINS
MANSA_QR
PREPAID_BALANCE
POSTPAID_ACCOUNT
SUBSCRIPTION
```

La présence d’un moyen dans le produit ne signifie pas qu’il est disponible sur chaque voie. La disponibilité réelle dépend de la configuration administrative et de l’état des équipements.

Le RFID UHF et la plaque ANPR sont des moyens d’identification, pas des boutons de paiement à afficher à l’usager.

## 9. État d’affichage de la borne

`GET /v1/access/terminals/:terminalId/display-state` fournit l’état minimal nécessaire à l’interface industrielle :

- état du service ;
- clé de titre ;
- clé d’instruction ;
- montant dû ;
- moyens disponibles ;
- mode QR ;
- site alternatif éventuel ;
- heure estimée de reprise ;
- date de génération.

L’écran ne doit jamais devenir la seule source de vérité d’une transaction. Il guide l’usager ; les lecteurs physiques, le contrôleur local et le backend appliquent les décisions sécurisées.

Les langues visibles sont configurables par exploitant. Pour le Mali, la configuration de référence doit au minimum prévoir français, bamanankan et anglais, avec langues supplémentaires possibles selon le site.

## 10. Espèces FCFA/XOF

Les billets et pièces sont traités séparément. Un événement `AccessCashValidationEvent` enregistre le terminal, la devise, la coupure, le type d’instrument, le résultat, l’heure et la corrélation.

Les résultats incluent notamment : accepté, coupure inconnue, suspect, endommagé, coupure désactivée ou cassette pleine.

La plateforme ne doit jamais qualifier automatiquement un billet de contrefait sans capacité matérielle et procédure métier explicitement validées. Un équipement peut signaler un instrument suspect ; les règles d’exploitation déterminent la suite.

Le profil de borne doit refléter les capacités réelles de rendu de monnaie et les coupures activées. Si le rendu devient impossible, le service peut désactiver les espèces ou imposer un mode compatible avec la configuration locale avant que l’usager n’insère son argent.

## 11. Disponibilité et mode dégradé

La disponibilité d’un site ou d’une voie peut être :

```text
ACTIVE
SUSPENDED
MAINTENANCE
DEGRADED
CLOSED
DISABLED
```

L’état doit être calculé à partir de la politique de l’exploitant et de la santé des équipements : RFID, caméra ANPR, TPE, passerelle Mobile Money, validateur billets, monnayeur, recycleurs, scanner QR, barrière, contrôleur, capteurs, imprimante, écran, interphone et réseau.

Une panne d’un moyen de paiement ne doit pas nécessairement fermer la voie si d’autres moyens compatibles restent disponibles. L’interface doit masquer ou désactiver uniquement les capacités réellement indisponibles et afficher un message public clair.

## 12. Personnalisation exploitant

La même plateforme doit supporter :

- État ou ministère ;
- agence routière ;
- concessionnaire autoroutier privé ;
- entreprise ;
- parking ;
- station-service ;
- campus ;
- réseau de transport.

Chaque organisation peut définir son logo, nom, couleurs, co-branding, langues, textes, tarifs, moyens de paiement et règles de disponibilité. Cette personnalisation ne doit jamais modifier les invariants de sécurité ou les contrats partagés.

## 13. Sécurité

Les routes de mutation sont protégées par authentification de workload/utilisateur selon le contexte, autorisation explicite, isolation multi-tenant, journal d’audit et moindre privilège.

Les décisions sensibles ne doivent jamais être prises uniquement par le client graphique de la borne. Le terminal ne doit recevoir que les secrets strictement nécessaires à son rôle, avec rotation et stockage matériel sécurisé lorsque possible.

Les commandes offline doivent être signées ou protégées contre altération, rejouées avec idempotence et rapprochées au retour du réseau.

## 14. Tests contractuels obligatoires

Le package `@mansa/contracts` doit conserver des tests runtime qui vérifient :

- toutes les valeurs publiées acceptées par les type guards ;
- rejet des valeurs inconnues ;
- alignement entre noms de routes et méthodes HTTP ;
- maintien du préfixe `/v1/access` ;
- présence des capacités critiques péage : RFID UHF, ANPR, carte, QR MANSA, billets, pièces, double hauteur, recycleurs, reçu et interphone.

Ces tests protègent le contrat contre les suppressions accidentelles lors du développement progressif.

## 15. Prochaine tranche technique

Après verrouillage du contrat partagé, l’implémentation backend doit progresser dans cet ordre :

1. modèles Prisma pour credentials, entitlements, terminaux, disponibilité et usages ;
2. repositories multi-tenant ;
3. moteur de décision pur et tests négatifs ;
4. routes internes protégées ;
5. adaptateur matériel mock déterministe ;
6. persistance PostgreSQL et tests d’intégration ;
7. contrôleur local/offline et synchronisation idempotente ;
8. adaptateurs fabricants réels uniquement après validation contractuelle et matérielle.
