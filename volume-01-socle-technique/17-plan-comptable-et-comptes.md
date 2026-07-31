# Plan comptable et comptes du grand livre

## Objectif

Le plan comptable structure les comptes utilisés par le grand livre Mansa. Une écriture ne référence jamais directement un utilisateur, un commerçant ou un partenaire sans passer par un compte comptable identifié, typé, mono-devise et gouverné.

La primitive actuelle est `LedgerAccount`, exposée par `@mansa/domain` depuis `packages/domain/src/ledger-account.ts`.

## Types de comptes

Le socle reconnaît cinq familles :

- `ASSET` : actifs détenus ou à recevoir, par exemple comptes bancaires de règlement ;
- `LIABILITY` : dettes envers les utilisateurs, commerçants ou partenaires ;
- `EQUITY` : capitaux propres et comptes assimilés ;
- `REVENUE` : produits, notamment commissions et frais ;
- `EXPENSE` : charges, remboursements de frais et coûts opérationnels.

Le rattachement définitif au plan comptable réglementaire doit être validé avec la banque partenaire et les professionnels comptables compétents.

## Structure canonique

Un compte comporte :

- `id` : identifiant technique immuable ;
- `code` : code unique lisible et normalisé en majuscules ;
- `name` : libellé fonctionnel ;
- `type` : famille comptable ;
- `currency` : devise ISO 4217 unique ;
- `status` : `ACTIVE`, `FROZEN` ou `CLOSED` ;
- `ownerReference` : référence métier facultative, sans donnée personnelle sensible ;
- `createdAt` : date de création conservée en UTC.

## Convention de codes

Les codes contiennent de 2 à 80 caractères parmi les lettres, chiffres, deux-points, tirets et tirets bas.

Exemples :

```text
WALLET:USER-123
WALLET:MERCHANT-456
BANK:SETTLEMENT:XOF
REVENUE:PAYMENT_FEES:XOF
SUSPENSE:PAYMENTS:XOF
```

Le code est une clé fonctionnelle stable. Un changement de libellé ne doit pas entraîner de changement de code. Une migration de plan comptable doit être versionnée et auditée.

## Mono-devise

Chaque compte est rattaché à une seule devise. Une opération multidevise est représentée par plusieurs écritures ou plusieurs groupes comptables liés par une opération de change, jamais par un compte contenant plusieurs devises.

## États du compte

### ACTIVE

Le compte accepte de nouvelles lignes comptables.

### FROZEN

Le compte est temporairement bloqué. Aucune nouvelle écriture métier ne doit y être comptabilisée, sauf opération technique explicitement autorisée et auditée, comme une contre-passation contrôlée.

### CLOSED

Le compte est définitivement fermé aux nouvelles écritures. Sa conservation reste obligatoire pour l’historique et l’audit.

La méthode `canPost()` de la primitive retourne `true` uniquement pour un compte actif. La couche de persistance doit aussi vérifier le statut dans la même transaction que l’écriture.

## Règles d’intégrité

- Un identifiant, un code et un nom non vides sont obligatoires.
- Le code respecte la convention canonique et est unique en base.
- Le type appartient au catalogue autorisé.
- Le statut appartient au catalogue autorisé.
- La devise est un code ISO 4217 sur trois lettres.
- La date de création doit être valide.
- Le compte est mono-devise.
- Les comptes ne sont jamais supprimés après utilisation.
- Un compte fermé ne peut pas être rouvert sans procédure administrative versionnée.

## Relation avec les écritures

Avant de persister une `JournalEntry`, le service comptable doit :

1. charger tous les comptes référencés ;
2. vérifier leur existence et leur statut ;
3. vérifier que leur devise correspond à celle de l’écriture ;
4. appliquer les autorisations propres aux comptes techniques ;
5. persister l’écriture et les projections dans une transaction atomique.

La primitive `JournalEntry` garantit l’équilibre et la mono-devise de l’écriture. La validation de l’existence des comptes et de leur état relève du service de domaine et du dépôt persistant.

## Sécurité et gouvernance

- Seuls les rôles financiers autorisés créent ou gèlent des comptes techniques.
- La création automatique d’un wallet suit une règle métier versionnée.
- Toute modification de statut est auditée avec auteur, motif, corrélation et horodatage.
- Aucun numéro bancaire réel, secret ou document KYC n’est stocké dans le code du plan comptable.
- Les comptes de cantonnement, règlement, revenus, suspens et wallets sont séparés.

## Critères d’acceptation

- Un compte valide normalise son code et sa devise en majuscules.
- Le statut par défaut est `ACTIVE`.
- Une référence propriétaire vide est remplacée par une valeur absente.
- Les champs obligatoires vides sont rejetés.
- Un code contenant des espaces ou caractères non autorisés est rejeté.
- Un type ou statut inconnu est rejeté à l’exécution.
- Une devise invalide est rejetée.
- Une date invalide est rejetée.
- `canPost()` refuse les comptes gelés et fermés.
- Les tests automatisés couvrent ces invariants.
