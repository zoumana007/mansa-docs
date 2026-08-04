# Catalogue API — bénéficiaires

## 1. Portée

Ce catalogue décrit les routes communes de gestion des bénéficiaires Mansa. Les chemins, méthodes et types partagés sont déclarés dans `mansa-platform/packages/contracts/src/beneficiary-api.ts` et `beneficiary.ts`.

Préfixe : `/v1/beneficiaries`.

Un bénéficiaire représente une destination enregistrée par un utilisateur pour accélérer un futur transfert ou paiement. L’enregistrement ne garantit jamais qu’une opération sera autorisée : chaque transaction reste soumise aux contrôles de solde, limites, conformité, fraude, disponibilité du partenaire et authentification forte.

## 2. Création

### `POST /v1/beneficiaries`

Crée un bénéficiaire à partir de `CreateBeneficiaryCommand` et retourne un `Beneficiary`.

Règles minimales :

- exiger une clé d’idempotence ;
- déduire l’utilisateur propriétaire depuis la session et vérifier la concordance avec la commande ;
- normaliser le pays, la devise et les identifiants de destination ;
- valider les champs obligatoires selon le type de bénéficiaire ;
- ne jamais stocker ni retourner un numéro de compte ou téléphone complet dans le contrat public ;
- détecter les doublons actifs pour une même destination ;
- placer le bénéficiaire en `PENDING_VERIFICATION` lorsque la politique l’exige ;
- appliquer les règles de conformité, sanctions, fraude et liste de blocage ;
- journaliser l’origine, l’appareil et le contexte de création.

Les types partagés sont `MANSA_USER`, `BANK_ACCOUNT`, `MOBILE_MONEY`, `MERCHANT` et `PUBLIC_SERVICE`.

## 3. Consultation

### `GET /v1/beneficiaries`

Liste les bénéficiaires du propriétaire avec pagination et filtres : type, statut, confiance, favori et recherche textuelle.

Règles minimales :

- le propriétaire est dérivé de la session pour un client ;
- un administrateur doit disposer d’une permission dédiée et d’un motif auditable ;
- les destinations restent masquées ;
- les entrées archivées peuvent être exclues par défaut ;
- la recherche ne doit pas exposer de bénéficiaires appartenant à un autre utilisateur ;
- l’ordre peut privilégier favoris et dernière utilisation sans modifier les règles de sécurité.

### `GET /v1/beneficiaries/:beneficiaryId`

Retourne un bénéficiaire autorisé. Une ressource hors périmètre ne doit pas révéler son existence.

## 4. Mise à jour

### `PATCH /v1/beneficiaries/:beneficiaryId`

Met à jour uniquement les propriétés non sensibles prévues par `UpdateBeneficiaryCommand`, notamment le nom affiché, le surnom et le statut favori.

Règles minimales :

- exiger une clé d’idempotence ;
- vérifier la concordance entre l’identifiant du chemin et celui de la commande ;
- interdire la modification directe de la destination ;
- créer un nouveau bénéficiaire lorsqu’une destination change ;
- limiter et assainir les textes saisis ;
- journaliser l’ancien état et le nouvel état ;
- ne pas réactiver implicitement une entrée bloquée ou archivée.

## 5. Vérification

### `POST /v1/beneficiaries/:beneficiaryId/verification`

Vérifie un bénéficiaire avec `VerifyBeneficiaryCommand` et retourne son état courant.

Les méthodes partagées sont `OTP`, `PIN`, `BIOMETRIC`, `STRONG_AUTHENTICATION` et `MANUAL_REVIEW`.

Règles minimales :

- exiger une clé d’idempotence ;
- choisir la méthode selon le risque et les capacités du canal ;
- vérifier l’expiration, le nombre d’essais et l’unicité du challenge ;
- ne jamais journaliser le code de vérification ;
- empêcher la réutilisation d’un challenge consommé ;
- imposer une revue manuelle lorsque la politique de risque l’exige ;
- marquer `trusted` uniquement selon une politique explicite ;
- tracer la méthode, le résultat et le niveau d’authentification sans donnée secrète.

La biométrie reste une preuve locale fournie par le système d’exploitation ou un composant certifié. Les données biométriques brutes ne sont pas transmises à Mansa.

## 6. Blocage et archivage

### `PATCH /v1/beneficiaries/:beneficiaryId/status`

Applique `BLOCKED` ou `ARCHIVED` avec `ChangeBeneficiaryStatusCommand`.

Règles minimales :

- exiger une clé d’idempotence ;
- demander un motif lorsque le changement résulte d’une action administrative ou de risque ;
- différencier le blocage de sécurité de l’archivage volontaire ;
- empêcher l’utilisation immédiate d’un bénéficiaire bloqué ou archivé ;
- ne pas supprimer les références nécessaires à l’audit et à l’historique ;
- notifier le propriétaire selon la cause et la politique de sécurité ;
- exiger une procédure dédiée pour toute future réactivation.

## 7. Protection contre la fraude

- Une création ou modification sensible peut déclencher une période de refroidissement configurable.
- Un nouveau bénéficiaire ne contourne jamais les limites transactionnelles.
- Les changements d’appareil, de numéro, de mot de passe ou de niveau KYC peuvent renforcer les contrôles.
- Les signaux de risque incluent fréquence, montant, destination inhabituelle, échec répété, appareil compromis et compte récemment récupéré.
- Les listes de blocage et décisions partenaires sont appliquées avant toute transaction.
- Une destination bloquée globalement ne doit pas pouvoir être réenregistrée sous un autre libellé.
- Les actions administratives sensibles respectent la séparation des tâches.

## 8. Confidentialité

- Les numéros de compte et téléphones sont masqués dans les réponses.
- Les valeurs complètes nécessaires au routage sont chiffrées et conservées hors des journaux applicatifs.
- Les noms et surnoms ne doivent pas être utilisés comme preuve d’identité.
- Les exports massifs sont interdits sans permission et justification dédiées.
- La rétention respecte les obligations applicables et les besoins d’audit.
- Les applications ne doivent pas mettre les destinations sensibles en cache non chiffré.

## 9. Cohérence technique

La source canonique est constituée de :

- `BENEFICIARY_API_ROUTES` ;
- `BENEFICIARY_API_METHODS` ;
- `BeneficiaryApiContract` ;
- `ListBeneficiariesQuery` ;
- les types et gardes du fichier `beneficiary.ts`.

Le paquet `@mansa/contracts` expose le catalogue depuis son index principal. Les contrôleurs NestJS, applications mobiles et portails doivent importer ces contrats au lieu de redéfinir les chemins, statuts ou charges utiles.

## 10. Critères d’acceptation

- Chaque route documentée possède la même méthode et le même chemin dans le catalogue TypeScript.
- Une même clé d’idempotence ne crée pas plusieurs bénéficiaires ni plusieurs vérifications.
- Un utilisateur ne peut consulter ou modifier que ses propres bénéficiaires.
- Une destination sensible n’est jamais retournée en clair.
- Les champs obligatoires sont validés selon le type de destination.
- Une destination ne peut pas être modifiée silencieusement.
- Un bénéficiaire bloqué ou archivé ne peut pas être utilisé.
- Les challenges expirés, consommés ou trop souvent échoués sont refusés.
- Les contrôles transactionnels restent appliqués après vérification du bénéficiaire.
- Toute action sensible enregistre acteur, contexte, résultat et horodatage sans secret.
