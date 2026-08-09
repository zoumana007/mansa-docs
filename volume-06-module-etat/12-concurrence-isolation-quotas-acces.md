# Concurrence et isolation des quotas — accès et mobilité

## 1. Objet

Cette tranche renforce la validation PostgreSQL du moteur d’accès et mobilité sur deux propriétés critiques :

1. la résistance à une rafale de requêtes concurrentes supérieure au quota disponible ;
2. l’isolation stricte des compteurs lorsqu’un même identifiant métier est réutilisé par plusieurs organisations.

Elle complète `11-validation-postgresql-acces-mobilite.md` et s’appuie sur `apps/api-gateway/test/access-postgres.test.mjs` dans `mansa-platform`.

## 2. Rafale concurrente

Le scénario de référence configure :

```text
organizationId = org-burst
entitlementId = entitlement-burst
period = DAY
maxUsesPerPeriod = 3
requêtes concurrentes = 12
```

Les douze réservations sont émises simultanément avec des `requestId` distincts.

Résultat obligatoire :

- exactement trois appels retournent `true` ;
- exactement neuf appels retournent `false` ;
- le compteur PostgreSQL final vaut `used = 3` ;
- la limite persistée vaut `limit = 3` ;
- exactement trois réservations sont persistées.

Cette validation complète le scénario minimal à deux transactions et vérifie que l’invariant tient lors d’une contention plus forte.

## 3. Isolation multi-tenant des compteurs

Les identifiants techniques fournis par des intégrateurs, exploitants ou systèmes externes ne sont pas supposés être globalement uniques entre organisations.

Le scénario de référence réutilise simultanément :

```text
entitlementId = shared-entitlement
requestId = shared-request
```

pour deux organisations différentes :

```text
org-quota-a
org-quota-b
```

Chaque organisation possède son propre quota de une utilisation.

Résultat obligatoire :

- la réservation du tenant A réussit ;
- la réservation du tenant B réussit ;
- deux compteurs indépendants existent ;
- chaque compteur reste à `used = 1` et `limit = 1` ;
- deux réservations distinctes sont persistées.

Aucune collision inter-tenant n’est acceptable sur les clés métier réutilisables.

## 4. Clés d’isolation de référence

Les compteurs sont isolés au minimum par :

```text
organizationId + entitlementId + periodStart
```

Les réservations idempotentes sont isolées au minimum par :

```text
organizationId + entitlementId + periodStart + requestId
```

L’`organizationId` fait donc partie de toutes les clés de concurrence et d’idempotence concernées par les quotas.

## 5. Pourquoi cette validation est nécessaire

Les systèmes de péage, contrôle d’accès, stationnement, transport ou services publics peuvent produire des pics simultanés :

- ouverture d’une barrière après arrivée groupée de véhicules ;
- validation de badges à l’entrée d’un site ;
- synchronisation de terminaux après retour du réseau ;
- replays techniques d’un partenaire ;
- reprise d’une file locale hors ligne.

Le quota ne doit jamais être dépassé à cause de la concurrence, et une activité d’une organisation ne doit jamais consommer le quota d’une autre.

## 6. Exécution

Les scénarios sont intégrés au test PostgreSQL :

```text
apps/api-gateway/test/access-postgres.test.mjs
```

Ils sont exécutés dans la CI via :

```bash
pnpm --filter @mansa/api-gateway test:postgres
```

La base utilisée par la CI reste temporaire et ne contient aucune donnée réelle ni secret de production.

## 7. Critères d’acceptation

Cette tranche est acceptée lorsque :

- une rafale de douze requêtes concurrentes avec un quota de trois n’autorise que trois consommations ;
- le compteur final ne dépasse jamais la limite ;
- le nombre de réservations persistées correspond exactement aux succès ;
- deux tenants réutilisant le même `entitlementId` et le même `requestId` disposent de compteurs indépendants ;
- aucun test n’utilise de donnée personnelle ou identifiant de production.

## 8. Suite

Les validations encore nécessaires pour fermer la tranche PostgreSQL accès/mobilité sont notamment :

- démontrer le rollback atomique du compteur lorsqu’une création de réservation échoue ;
- borner et tester explicitement la stratégie de retry sur conflit de sérialisation PostgreSQL ;
- tester la résolution persistée complète credential + entitlement + disponibilité + terminal ;
- tester le contrôleur HTTP interne avec `InternalServiceGuard` actif ;
- mesurer les index et temps de réponse avec un volume représentatif.
