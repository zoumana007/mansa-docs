# Volume 02 — Application Client

# Chapitre 05 — Bénéficiaires et destinataires

## 1. Objet

Le module Bénéficiaires permet à un client d'enregistrer, vérifier, retrouver et administrer les destinataires utilisés pour les transferts et paiements récurrents.

Il couvre les destinataires internes Mansa, comptes bancaires, portefeuilles Mobile Money, commerçants et services publics.

## 2. Principes de sécurité

- Aucun numéro de compte ou téléphone complet n'est renvoyé dans les listes courantes.
- Un nouveau bénéficiaire commence en `PENDING_VERIFICATION`.
- L'activation nécessite une méthode de vérification autorisée par la politique de risque.
- Une opération sensible peut exiger une nouvelle authentification forte même pour un bénéficiaire déjà vérifié.
- Le blocage et l'archivage sont conservés dans l'historique d'audit.
- La création utilise une clé d'idempotence pour éviter les doublons.

## 3. Types de bénéficiaires

Le contrat partagé définit les types suivants :

- `MANSA_USER` : utilisateur ou portefeuille interne ;
- `BANK_ACCOUNT` : compte bancaire externe ;
- `MOBILE_MONEY` : portefeuille d'un opérateur Mobile Money ;
- `MERCHANT` : commerçant Mansa ;
- `PUBLIC_SERVICE` : administration, université ou autre service public.

## 4. Cycle de vie

1. Le client saisit ou sélectionne le destinataire.
2. L'application résout et masque les coordonnées sensibles.
3. L'API crée un bénéficiaire en attente de vérification.
4. Le moteur de risque choisit la méthode de vérification.
5. Après validation, le bénéficiaire passe à `ACTIVE`.
6. Le client peut le marquer comme favori, le bloquer ou l'archiver.

Les statuts normalisés sont : `PENDING_VERIFICATION`, `ACTIVE`, `BLOCKED` et `ARCHIVED`.

## 5. Vérification

Les méthodes prévues sont :

- `OTP` ;
- `PIN` ;
- `BIOMETRIC` ;
- `STRONG_AUTHENTICATION` ;
- `MANUAL_REVIEW`.

La méthode effectivement utilisée dépend du montant, du canal, du niveau KYC, de l'appareil, du pays, du partenaire et des signaux de fraude.

## 6. Contrats techniques

La source de vérité initiale est :

```text
mansa-platform/packages/contracts/src/beneficiary.ts
```

Elle expose les modèles :

- `BeneficiaryDestination` ;
- `Beneficiary` ;
- `CreateBeneficiaryCommand` ;
- `VerifyBeneficiaryCommand` ;
- `UpdateBeneficiaryCommand` ;
- `ChangeBeneficiaryStatusCommand`.

Les catalogues et gardes de type sont couverts par des tests dans :

```text
mansa-platform/packages/contracts/test/beneficiary.test.mjs
```

Le contrat est exporté depuis l'entrée principale `@mansa/contracts` et depuis le sous-chemin public dédié :

```ts
import type { Beneficiary } from '@mansa/contracts/beneficiary';
```

Le sous-chemin est déclaré dans `packages/contracts/package.json` et pointe vers les fichiers générés `dist/beneficiary.js` et `dist/beneficiary.d.ts`.

## 7. API cible

| Route | Usage |
|---|---|
| `GET /beneficiaries` | Lister et filtrer les bénéficiaires |
| `POST /beneficiaries` | Créer un bénéficiaire |
| `POST /beneficiaries/{id}/verify` | Vérifier et activer |
| `PATCH /beneficiaries/{id}` | Modifier le nom, surnom ou favori |
| `POST /beneficiaries/{id}/block` | Bloquer |
| `POST /beneficiaries/{id}/archive` | Archiver |
| `GET /beneficiaries/{id}/eligibility` | Vérifier les canaux et limites disponibles |

## 8. Règles d'interface

- La liste affiche le nom, le type, un identifiant masqué, le statut et la dernière utilisation.
- Les favoris sont affichés en premier sans modifier les résultats de recherche.
- Un bénéficiaire bloqué ou archivé ne peut pas être sélectionné pour une nouvelle opération.
- L'application explique clairement pourquoi une vérification supplémentaire est demandée.
- Toute erreur de résolution d'un compte externe doit rester neutre afin de ne pas révéler l'existence d'un compte à un tiers.

## 9. Évolutions prévues

Les prochains lots devront ajouter les schémas de validation d'entrée, le stockage backend, la résolution des coordonnées partenaires, l'évaluation de risque et l'intégration au parcours de transfert.
