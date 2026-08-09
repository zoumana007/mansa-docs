# Validation PostgreSQL — accès et mobilité

## 1. Objet

Cette tranche consolide la persistance du moteur d’accès et mobilité par des tests d’intégration exécutés contre PostgreSQL réel dans la CI de `mansa-platform`.

Elle complète :

- `09-schema-postgresql-acces-mobilite.md` ;
- `10-persistance-prisma-service-http-acces.md`.

Le but est de vérifier les propriétés qui ne peuvent pas être garanties uniquement par des tests unitaires : contraintes uniques multi-tenant, idempotence réellement persistée, isolation transactionnelle et concurrence sur les quotas.

## 2. Fichier de test de référence

Le dépôt plateforme contient :

```text
apps/api-gateway/test/access-postgres.test.mjs
```

Ce fichier est opt-in hors CI via :

```text
RUN_POSTGRES_TESTS=1
```

et nécessite `DATABASE_URL`.

Le script `apps/api-gateway/package.json` exécute désormais dans `test:postgres` les validations PostgreSQL du rapprochement financier et de l’accès/mobilité.

## 3. Isolation multi-tenant du requestId

La contrainte de décision est :

```text
organizationId + requestId
```

Deux organisations différentes peuvent donc recevoir le même identifiant de requête sans collision.

Le test crée deux décisions portant le même `requestId` pour deux tenants puis vérifie :

- deux lignes persistées ;
- une relecture indépendante par tenant ;
- aucune fuite de corrélation entre organisations.

Cette propriété est obligatoire pour les bornes, opérateurs et partenaires qui peuvent générer localement leurs propres identifiants.

## 4. Idempotence d’une décision

`recordDecision()` utilise un `upsert` dont la branche `update` est vide.

La première décision validée pour :

```text
organizationId + requestId
```

reste donc la référence. Un retry ne doit pas réécrire silencieusement la décision initiale avec un résultat différent.

Le test persiste une première décision puis tente une seconde écriture contradictoire avec la même clé. La relecture doit restituer exclusivement la première décision.

## 5. Dernière unité d’un quota

La réservation de quota s’exécute dans une transaction PostgreSQL `SERIALIZABLE`.

Le scénario concurrent de référence est :

```text
quota = 1
requête A || requête B
```

Les deux requêtes visent le même tenant, le même entitlement et la même fenêtre temporelle.

Résultat attendu :

- une seule réservation réussit ;
- l’autre reçoit `false` ;
- le compteur final vaut exactement `1` ;
- une seule réservation est persistée.

Aucun dépassement de quota n’est accepté même si les deux transactions démarrent simultanément.

## 6. Replay concurrent de la même réservation

Un second scénario lance deux appels concurrents avec exactement le même :

```text
organizationId
entitlementId
periodStart
requestId
```

Les deux appels doivent être considérés comme le même événement métier.

Résultat attendu :

- les deux appels retournent un succès idempotent ;
- le compteur n’est incrémenté qu’une seule fois ;
- une seule réservation existe en base.

Les collisions uniques et conflits de sérialisation sont donc transformés en replay contrôlé lorsque la réservation gagnante existe déjà.

## 7. CI

Le workflow :

```text
.github/workflows/ci.yml
```

démarre PostgreSQL 17, applique les migrations Prisma, exécute les validations générales puis lance :

```bash
pnpm --filter @mansa/api-gateway test:postgres
```

Cette commande couvre désormais au minimum :

- rapprochement financier PostgreSQL ;
- accès/mobilité PostgreSQL.

Aucun secret réel n’est nécessaire. La base CI utilise uniquement des identifiants temporaires locaux au job GitHub Actions.

## 8. Propriétés validées par cette tranche

La tranche verrouille désormais :

1. l’isolation de `requestId` par organisation ;
2. la relecture idempotente d’une décision persistée ;
3. l’absence de double consommation de la dernière unité de quota ;
4. le replay concurrent idempotent d’une même réservation ;
5. l’exécution automatique de ces scénarios dans la CI PostgreSQL.

## 9. Points restant à valider

La validation PostgreSQL du module accès/mobilité n’est pas encore exhaustive.

Restent notamment :

- provoquer explicitement un échec de création de réservation et démontrer le rollback du compteur dans PostgreSQL ;
- tester le contrôleur HTTP interne avec `InternalServiceGuard` réellement actif ;
- tester la résolution complète credential + entitlement + disponibilité + terminal avec données persistées ;
- tester le comportement lors de fortes concurrences répétées au-delà de deux transactions ;
- vérifier les performances et index avec un volume représentatif ;
- ajouter les API publiques administratives de gestion des credentials, droits et états de borne une fois ces invariants stabilisés.

## 10. Critères d’acceptation

Cette tranche est acceptée lorsque :

- la CI applique les migrations avant les tests ;
- `test:postgres` exécute les tests du rapprochement et de l’accès ;
- deux tenants partageant un `requestId` restent isolés ;
- une décision rejouée reste identique à la première décision persistée ;
- deux consommateurs concurrents ne dépassent pas un quota de un ;
- deux replays concurrents de la même réservation ne consomment qu’une unité ;
- aucun identifiant réel ni secret de production n’est introduit dans les fixtures.
