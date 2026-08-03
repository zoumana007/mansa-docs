# Catalogue API — transferts

## 1. Portée

Ce catalogue décrit les routes publiques du domaine transfert. Les chemins, méthodes et types partagés sont déclarés dans `mansa-platform/packages/contracts/src/transfer-api.ts`.

Préfixe : `/v1`.

Le parcours sépare explicitement le devis, la création, l’autorisation et l’exécution. Cette séparation permet d’appliquer les frais, limites, contrôles de conformité et exigences d’authentification renforcée avant tout mouvement de fonds.

## 2. Routes

### `POST /v1/transfers/quotes`

Calcule un devis à partir de `QuoteTransferCommand` et retourne un `TransferQuote`.

Le devis doit inclure le type de transfert, le montant, les frais détaillés et une date d’expiration. Il ne réserve ni ne débite les fonds.

Règles minimales :

- vérifier l’accès au portefeuille source ;
- vérifier le bénéficiaire et sa capacité à recevoir le transfert ;
- appliquer les limites, frais, taxes et règles du pays ;
- sélectionner le canal ou partenaire compatible ;
- refuser toute incohérence de devise ;
- limiter la durée de validité du devis.

### `POST /v1/transfers`

Crée un transfert à partir de `CreateTransferCommand` et retourne un `Transfer`.

La commande référence un devis non expiré et exige une clé d’idempotence. La création doit figer les paramètres financiers acceptés par le client afin qu’une modification ultérieure de tarification ne change pas l’opération en cours.

### `GET /v1/transfers/:transferId`

Retourne le transfert visible par son propriétaire ou par un rôle administratif autorisé dans le bon périmètre.

Une ressource hors périmètre ne doit pas révéler son existence. Les statuts et codes d’échec proviennent du contrat `transfer.ts`.

### `POST /v1/transfers/:transferId/authorize`

Autorise un transfert créé à partir de `AuthorizeTransferCommand`.

Le service vérifie que l’identifiant contenu dans le chemin correspond à celui de la commande, que l’acteur est bien le propriétaire et que la méthode d’authentification satisfait les obligations de risque. Un transfert déjà autorisé ou finalisé ne doit pas être débité une seconde fois.

### `POST /v1/transfers/:transferId/cancel`

Demande l’annulation d’un transfert à partir de `CancelTransferCommand` et retourne l’état résultant.

L’annulation n’est acceptée que tant que le transfert se trouve dans un état annulable. Une opération déjà transmise de manière irréversible à un partenaire passe par un mécanisme de rejet, de retour ou de compensation, jamais par une modification silencieuse.

## 3. Idempotence et concurrence

1. Une même clé et une même commande retournent le même transfert.
2. Une même clé avec un contenu différent est rejetée.
3. Une autorisation répétée ne déclenche jamais un second débit.
4. Une annulation concurrente avec l’exécution produit un seul état final cohérent.
5. Les tentatives partenaires conservent leur référence et leur résultat.
6. Un timeout ne doit pas être interprété comme un échec définitif sans réconciliation.

## 4. Cycle de vie

Les statuts possibles sont définis par `TRANSFER_STATUSES`. Les transitions sont pilotées par le service métier et doivent respecter une machine à états explicite.

Les états finaux `COMPLETED`, `FAILED`, `CANCELLED` et `REVERSED` ne sont pas modifiés directement. Toute correction financière utilise une opération compensatrice auditée.

## 5. Sécurité et conformité

- Authentification obligatoire pour toutes les routes.
- Autorisation par propriétaire, portefeuille, bénéficiaire, organisation, pays et environnement.
- Authentification renforcée selon montant, canal, appareil et score de risque.
- Contrôles KYC, sanctions, fraude et limites avant exécution.
- Masquage des données du bénéficiaire dans les journaux.
- Corrélation et audit de chaque étape.
- Protection anti-rejeu des callbacks et webhooks partenaires.
- Blocage immédiat possible par configuration administrative.

## 6. Cohérence technique

La source canonique est constituée de :

- `TRANSFER_API_ROUTES`
- `TRANSFER_API_METHODS`
- `TransferApiContract`
- `QuoteTransferCommand`
- `CreateTransferCommand`
- `AuthorizeTransferCommand`
- `CancelTransferCommand`

Le paquet `@mansa/contracts` expose le catalogue via le sous-chemin `@mansa/contracts/transfer-api`. Les contrôleurs NestJS et les clients mobiles ou web doivent importer ce contrat au lieu de dupliquer les routes.

## 7. Critères d’acceptation

- Chaque route documentée possède la même méthode et le même chemin dans le catalogue TypeScript.
- Un devis expiré ne permet pas de créer un transfert.
- Toute création utilise une clé d’idempotence.
- L’autorisation vérifie le propriétaire et le niveau d’authentification requis.
- Un transfert ne peut être débité qu’une seule fois.
- Les montants et frais conservent leur précision et leur devise.
- Les états finaux sont immuables.
- Les erreurs sont structurées et corrélées sans divulgation de données sensibles.
- Les transferts incertains sont rapprochés avec le partenaire avant toute nouvelle tentative financière.
