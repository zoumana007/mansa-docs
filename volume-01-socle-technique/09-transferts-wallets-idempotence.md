# Transferts entre wallets et idempotence

## 1. Objet

Cette spécification décrit les règles obligatoires pour un transfert interne entre deux wallets Mansa. Elle correspond au noyau métier implémenté dans `mansa-platform/packages/domain` et prépare son branchement au grand livre persistant.

## 2. Commande de transfert

Une commande contient au minimum :

- `transferId` : identifiant métier unique du transfert ;
- `idempotencyKey` : clé fournie par le client ou générée à la frontière API ;
- `sourceWalletId` : wallet débité ;
- `destinationWalletId` : wallet crédité ;
- `amount` : montant positif en unités mineures ;
- `currency` : devise conforme au wallet et au produit ;
- métadonnées autorisées et validées.

Les identifiants textuels sont normalisés avant comparaison. Une valeur vide après normalisation est invalide.

## 3. Invariants métier

1. Le wallet source et le wallet destination doivent être différents après normalisation.
2. Le montant doit être strictement positif.
3. Le montant ne doit jamais être représenté par un nombre flottant.
4. La devise doit être explicite et compatible avec les deux wallets.
5. La date de complétion doit être valide avant toute écriture.
6. Une date reçue ou exposée par un objet métier doit être copiée afin qu’une mutation externe ne puisse pas modifier l’historique du transfert.
7. L’exécution atomique doit retourner un objet contenant un `transactionId` de type chaîne, non vide après normalisation.
8. Un résultat d’exécution nul, incomplet ou d’un type inattendu doit être refusé avant l’enregistrement du résultat idempotent.
9. Les métadonnées doivent être validées avant toute mutation financière.
10. Un échec de validation ne doit produire aucune écriture ni événement métier.

## 4. Idempotence

### 4.1 Premier appel

Lorsque la clé d’idempotence n’existe pas, le service exécute la mutation atomique puis enregistre la commande normalisée et son résultat.

### 4.2 Rejeu identique

Lorsque la même clé existe avec une commande équivalente, le service renvoie le résultat déjà enregistré sans débiter ni créditer une seconde fois.

### 4.3 Conflit

Lorsque la même clé existe avec une commande différente, le service refuse l’opération avec une erreur de conflit. Aucune nouvelle mutation n’est autorisée.

### 4.4 Conservation

Les résultats d’idempotence des opérations financières doivent être conservés selon une politique validée par conformité et exploitation. La suppression automatique ne doit jamais rendre possible le rejeu d’un ancien débit encore pertinent.

## 5. Transaction persistante cible

L’adaptateur PostgreSQL devra effectuer dans une transaction unique :

1. lecture et contrôle de l’enregistrement d’idempotence ;
2. verrouillage ou contrôle concurrentiel des wallets ;
3. contrôle du statut, de la devise, des limites et du solde disponible ;
4. création de la transaction métier ;
5. écriture des lignes du grand livre en partie double ;
6. mise à jour des soldes projetés ;
7. enregistrement du résultat idempotent ;
8. insertion d’un événement dans l’outbox transactionnelle.

La publication vers le bus d’événements intervient après validation de la transaction de base de données.

## 6. États recommandés

- `PENDING` : commande acceptée mais mutation non finalisée ;
- `COMPLETED` : débit, crédit et écritures comptables validés ;
- `FAILED` : échec définitif sans mouvement partiel ;
- `REVERSED` : transfert compensé par une nouvelle écriture, sans suppression de l’historique.

Un transfert terminé ne doit jamais être modifié en place pour simuler une annulation.

## 7. Contrôles de sécurité et risque

Avant exécution, la couche applicative vérifie notamment :

- authentification et autorisation de l’acteur ;
- statut KYC et statut du compte ;
- restrictions pays, produit et partenaire ;
- limites par opération, jour et période ;
- blocages fraude, conformité ou administration ;
- cohérence du propriétaire ou du mandat utilisé ;
- absence de secret ou donnée KYC sensible dans les métadonnées libres.

## 8. Observabilité

Chaque tentative doit disposer d’un identifiant de corrélation. Les journaux peuvent contenir les identifiants techniques nécessaires au diagnostic, mais jamais de jeton, code secret, numéro de carte complet ou document KYC.

Les métriques minimales couvrent : volume, succès, échec, conflit d’idempotence, latence et nombre de rejouements.

## 9. Critères d’acceptation

- Une commande valide produit un seul transfert terminé.
- Deux appels identiques avec la même clé ne produisent qu’une mutation.
- Deux commandes différentes avec la même clé sont refusées.
- Des wallets identiques après normalisation sont refusés.
- Un identifiant de transaction vide est refusé.
- Un résultat d’exécution nul, incomplet ou non typé est refusé sans enregistrement.
- Une date invalide est refusée avant mutation.
- La mutation de la date fournie au constructeur ou renvoyée par l’objet ne modifie jamais la date interne sérialisée.
- Des métadonnées invalides sont refusées avant mutation.
- Les tests du paquet `@mansa/domain` réussissent.
- La documentation du paquet renvoie aux mêmes invariants.

## 10. Traçabilité code-documentation

- Code : `mansa-platform/packages/domain`.
- Documentation locale du paquet : `mansa-platform/packages/domain/README.md`.
- Tests : `mansa-platform/packages/domain/test/transfer-service.test.mjs` et tests associés.
- Prochaine étape technique : adaptateur de persistance et grand livre transactionnel dans l’API Gateway.
