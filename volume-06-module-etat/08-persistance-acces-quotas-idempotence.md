# Persistance des accès — quotas, périodes et idempotence

## 1. Objet

Cette tranche prépare l’implémentation PostgreSQL/Prisma du service applicatif d’accès en figeant deux éléments qui doivent rester identiques entre contrats, base de données et tests :

- le calcul des fenêtres de quota ;
- les identités idempotentes utilisées pour les décisions, usages et réservations.

L’implémentation de référence se trouve dans :

`mansa-platform/packages/contracts/src/access-persistence.ts`.

## 2. Fenêtres de quota

Les quotas `DAY`, `WEEK` et `MONTH` utilisent des fenêtres calendaires UTC semi-ouvertes :

```text
[startInclusive, endExclusive)
```

Règles :

- `DAY` : de `00:00:00.000Z` au jour suivant exclu ;
- `WEEK` : du lundi `00:00:00.000Z` au lundi suivant exclu ;
- `MONTH` : du premier jour du mois `00:00:00.000Z` au premier jour du mois suivant exclu.

Le stockage PostgreSQL devra utiliser des timestamps avec fuseau cohérents avec ces bornes. Une adaptation future à des périodes réglementaires locales ne doit pas modifier silencieusement cette convention : elle devra introduire une politique explicite et versionnée.

## 3. Pourquoi UTC

Le moteur d’accès peut être utilisé par plusieurs pays et plusieurs exploitants. Une fenêtre commune UTC évite qu’un changement de fuseau du terminal ou un passage heure d’été/heure d’hiver modifie rétroactivement la consommation d’un quota.

L’heure locale reste utilisable pour l’affichage, les rapports et les règles métier spécifiques, mais la clé technique de réservation de quota reste déterministe.

## 4. Identités idempotentes

Pour une requête donnée, les responsabilités sont séparées :

```text
access-decision:{organizationId}:{requestId}
access-usage:{organizationId}:{requestId}
access-quota:{organizationId}:{entitlementId}:{periodStart}:{requestId}
```

Ces valeurs sont des identités logiques. La base peut les stocker telles quelles ou les matérialiser avec des colonnes et contraintes uniques équivalentes.

Exigences minimales :

1. une seule décision finale par organisation et `requestId` ;
2. un seul usage par organisation et `requestId` ;
3. une seule réservation de quota par droit, fenêtre et requête ;
4. deux tenants différents peuvent utiliser le même `requestId` sans collision ;
5. une répétition réseau de la même requête ne consomme jamais deux places de quota.

## 5. Séparateurs et valeurs ambiguës

Le helper de référence rejette les segments vides et les identifiants contenant `:`. Cette règle évite qu’une concaténation produise deux identités textuelles ambiguës.

La future persistance Prisma peut préférer des contraintes composites natives plutôt que des chaînes concaténées. Dans ce cas, la sémantique doit rester strictement équivalente.

## 6. Concurrence PostgreSQL

Le calcul de fenêtre ne garantit pas à lui seul la concurrence. L’implémentation PostgreSQL suivante doit encore garantir atomiquement :

```text
compteur < limite
+ réservation de la requête absente
→ création de la réservation
→ succès
```

Deux transactions concurrentes sur le dernier emplacement disponible ne doivent jamais toutes les deux réussir.

Approches acceptables selon le schéma retenu :

- compteur matérialisé avec `UPDATE ... WHERE used < limit` ;
- verrou de ligne sur un compteur de période ;
- insertion de réservation avec contrainte unique puis comptage/verrou dans la même transaction ;
- autre mécanisme PostgreSQL démontré par test concurrent.

Un simple `SELECT count(*)` suivi d’un `INSERT` non protégé est interdit.

## 7. Tests de référence

La suite :

`mansa-platform/packages/contracts/test/access-persistence.test.mjs`

couvre :

- journée UTC ;
- semaine du lundi au lundi ;
- changement de mois et d’année ;
- génération déterministe des trois identités ;
- rejet des timestamps invalides ;
- rejet des segments ambigus.

## 8. Cohérence attendue avec Prisma

La prochaine tranche PostgreSQL devra utiliser ces mêmes règles pour les futurs modèles :

- credential ;
- entitlement ;
- service availability ;
- terminal profile ;
- access decision ;
- access usage ;
- quota reservation ou compteur de période.

Les contraintes uniques de la base doivent correspondre aux identités définies ici. Les tests d’intégration PostgreSQL devront réutiliser les bornes de période fournies par `calculateAccessQuotaWindow` au lieu de recalculer les dates dans le repository.

## 9. Étape suivante

La prochaine tranche doit maintenant matérialiser la persistance PostgreSQL/Prisma et implémenter les ports de `AccessApplicationRepository`, `AccessQuotaReservation` et `AccessDecisionJournal`, avec en priorité :

- isolation stricte par `organizationId` ;
- décision et usage idempotents ;
- réservation atomique du dernier quota disponible ;
- deux requêtes concurrentes réelles contre PostgreSQL ;
- reprise d’une requête déjà enregistrée après timeout ;
- absence de double action physique ou financière.
