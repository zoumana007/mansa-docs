# Persistance PostgreSQL du grand livre

## 1. Objet

Ce document définit le modèle de persistance du grand livre Mansa et aligne le schéma PostgreSQL du dépôt `mansa-platform` avec les contrats TypeScript de `packages/contracts/src/ledger.ts` et `packages/contracts/src/ledger-api.ts`.

Le schéma de référence est :

`mansa-platform/apps/api-gateway/prisma/schema.prisma`

Ce lot ne rend pas encore le ledger exploitable en production : il prépare la persistance, l’idempotence, la compensation, les projections de solde et l’outbox transactionnelle. Le service NestJS et les transactions SQL atomiques restent à implémenter.

## 2. Alignement des statuts

Les statuts persistés doivent correspondre exactement au contrat partagé :

- `PENDING` ;
- `POSTED` ;
- `REVERSED` ;
- `REJECTED`.

Le statut historique `FAILED` ne doit pas être utilisé pour une transaction comptable métier. Les échecs techniques sont représentés dans les journaux, métriques ou files de reprise, sans modifier rétroactivement une transaction `POSTED`.

## 3. Comptes comptables

Chaque `LedgerAccount` conserve :

- un identifiant UUID ;
- un code comptable unique ;
- le type de propriétaire : `PLATFORM`, `USER`, `MERCHANT`, `PARTNER` ou `PUBLIC_BODY` ;
- l’identifiant du propriétaire lorsqu’il existe ;
- le type comptable : actif, passif, capitaux propres, produit ou charge ;
- la devise ISO 4217 ;
- le pays ISO 3166-1 alpha-2 ;
- l’indicateur de compte système ;
- les dates de création et de mise à jour.

Un compte rattaché à un wallet conserve la relation existante, mais les comptes techniques de plateforme peuvent ne pas avoir de wallet.

## 4. Transactions comptables

Une `LedgerTransaction` persistée contient au minimum :

- `reference` ;
- `transactionType` ;
- `idempotencyKey` unique ;
- `requestFingerprint` ;
- `correlationId` ;
- `countryCode` ;
- `status` ;
- `occurredAt` ;
- `postedAt` lorsque la transaction est publiée ;
- les métadonnées JSON non sensibles ;
- le lien de compensation éventuel.

### Empreinte de requête

`requestFingerprint` est une empreinte cryptographique déterministe calculée sur la commande normalisée. Elle permet de distinguer :

1. la répétition exacte d’une requête avec la même clé d’idempotence, qui doit retourner le résultat existant ;
2. la réutilisation de la même clé avec un contenu différent, qui doit être rejetée en conflit.

L’empreinte ne doit jamais inclure de secret ni de donnée inutilement sensible.

## 5. Compensation

Une correction n’édite jamais les écritures originales.

La transaction compensatoire renseigne `reversalOfTransactionId` vers la transaction d’origine. Cette relation est unique afin qu’une transaction ne soit pas compensée plusieurs fois par erreur.

Le backend doit exécuter dans une même transaction SQL :

1. verrouillage de la transaction d’origine ;
2. contrôle du statut `POSTED` ;
3. contrôle de l’absence de compensation existante ;
4. création de la transaction compensatoire ;
5. création des écritures inversées ;
6. mise à jour de la projection des soldes ;
7. création de l’événement outbox ;
8. passage de la transaction d’origine à `REVERSED` ;
9. commit.

## 6. Écritures

`LedgerEntry` ajoute une `sequence` déterministe par transaction et un `postedAt` explicite.

Contraintes attendues :

- unicité `(transactionId, sequence)` ;
- montant strictement positif ;
- devise identique pour toutes les écritures d’une transaction ;
- somme des débits égale à la somme des crédits ;
- compte existant et compatible avec la devise ;
- impossibilité de supprimer une écriture publiée.

Les invariants nécessitant une vue globale de la transaction ne doivent pas reposer uniquement sur Prisma. Ils doivent être vérifiés par le service métier et, lorsque possible, renforcés par des contraintes ou procédures PostgreSQL.

## 7. Pagination stable

L’index `(accountId, postedAt, id)` fournit l’ordre stable attendu par le contrat de pagination par curseur.

Le curseur doit être construit à partir de la position `(postedAt, id)` de la dernière écriture retournée. Une requête suivante doit utiliser une comparaison keyset et non un `OFFSET`.

## 8. Projection de solde

`LedgerBalanceProjection` est une projection reconstruisible. Elle ne remplace jamais le grand livre comme source de vérité.

Elle conserve :

- `availableMinor` ;
- `pendingMinor` ;
- la devise ;
- le dernier `postedAt` appliqué ;
- le dernier identifiant d’écriture ;
- une `projectionSequence` monotone ;
- la date de dernière mise à jour.

### Règles

- une projection est mise à jour dans la même transaction SQL que les écritures ;
- un verrou pessimiste ou une stratégie de sérialisation empêche les mises à jour concurrentes perdues ;
- un job de réconciliation doit pouvoir recalculer la projection depuis les écritures ;
- tout écart déclenche une alerte et bloque les opérations présentant un risque financier.

## 9. Outbox transactionnelle

`OutboxEvent` stocke les événements à publier après le commit du ledger.

Champs principaux :

- type et identifiant d’agrégat ;
- type d’événement ;
- payload JSON ;
- statut `PENDING`, `PUBLISHED` ou `FAILED` ;
- compteur de tentatives ;
- date de prochaine disponibilité ;
- date de publication ;
- dernière erreur technique ;
- transaction comptable liée lorsqu’elle existe.

Le worker outbox doit :

1. sélectionner un lot `PENDING` disponible ;
2. verrouiller les lignes avec une stratégie compatible avec plusieurs workers ;
3. publier vers le bus cible ;
4. marquer l’événement `PUBLISHED` après confirmation ;
5. incrémenter `attempts` et reprogrammer en cas d’erreur ;
6. déplacer vers un traitement d’incident après dépassement du seuil configurable.

Le consommateur externe reste idempotent : la publication peut être effectuée au moins une fois.

## 10. Migration

Avant application sur une base existante :

1. sauvegarder la base ;
2. vérifier les données historiques utilisant `FAILED` ;
3. définir leur correspondance métier avant changement d’énumération ;
4. remplir les nouveaux champs obligatoires pour les transactions existantes ;
5. générer la migration Prisma dans un environnement de développement ;
6. relire le SQL généré ;
7. exécuter `prisma validate` ;
8. tester la migration sur une copie représentative ;
9. tester le rollback opérationnel ou la restauration ;
10. seulement ensuite déployer en Recette.

Aucune migration destructive automatique ne doit être lancée sur une base de production.

## 11. Tests obligatoires du prochain lot backend

Le module persistant doit couvrir au minimum :

- publication équilibrée ;
- rejet d’une transaction déséquilibrée ;
- rejet d’un montant nul ou négatif ;
- répétition d’une clé idempotente avec contenu identique ;
- conflit d’idempotence avec contenu différent ;
- transaction contenant plusieurs devises ;
- compte dans une devise incompatible ;
- compensation réussie ;
- double compensation rejetée ;
- concurrence de deux publications sur le même compte ;
- reconstruction de la projection de solde ;
- reprise d’un événement outbox après échec ;
- pagination stable des écritures.

## 12. État du lot

Le schéma Prisma contient désormais les structures nécessaires pour :

- aligner les comptes sur le contrat partagé ;
- persister les champs d’idempotence et de corrélation ;
- représenter les compensations ;
- ordonner les écritures ;
- maintenir une projection de solde reconstruisible ;
- enregistrer les événements dans une outbox transactionnelle.

Restent à construire avant que ce sous-système soit considéré terminé : la migration Prisma générée et testée, le `PrismaModule`, les repositories, le service ledger transactionnel, les contrôleurs internes, le worker outbox, la réconciliation, les métriques et les tests PostgreSQL/concurrence.
