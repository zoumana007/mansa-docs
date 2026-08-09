# Service applicatif d’accès Mansa

## 1. Objet

Cette tranche entoure le moteur de décision déterministe avec une orchestration applicative explicite. Elle ne remplace ni la persistance PostgreSQL ni les adaptateurs matériels, mais fixe le contrat entre résolution des données, décision, réservation de quota et journalisation.

L’implémentation de référence est :

`mansa-platform/packages/contracts/src/access-application-service.ts`.

## 2. Responsabilités

`processAccessRequest` exécute dans un ordre stable :

1. résolution du credential ;
2. chargement de l’état de service ;
3. chargement du profil terminal ;
4. vérification du périmètre organisation/site/terminal ;
5. résolution du droit lorsque le credential existe ;
6. lecture du compteur d’usage si un quota existe ;
7. appel de `evaluateAccessDecision` ;
8. réservation atomique du quota si la décision est `ALLOW` ;
9. conversion d’un échec concurrent de réservation en `DENY / USAGE_LIMIT_REACHED` ;
10. journalisation de la décision ;
11. journalisation de l’usage uniquement après autorisation définitive.

Le service ne déclenche pas lui-même une barrière, un débit bancaire, un TPE ou un périphérique industriel.

## 3. Dépendances injectées

La couche applicative dépend de trois ports.

### 3.1 `AccessApplicationRepository`

Il fournit :

- `resolveCredential` ;
- `resolveEntitlement` ;
- `loadServiceAvailability` ;
- `loadTerminalProfile` ;
- `countUsageInCurrentPeriod`.

L’implémentation de production de ce port devra être PostgreSQL/Prisma et appliquer systématiquement les filtres de tenant.

### 3.2 `AccessQuotaReservation`

La méthode `reserve` représente une réservation atomique de quota. Elle reçoit le droit, la période, la limite, la requête et la corrélation.

Le booléen retourné signifie :

- `true` : la place de quota est réservée pour cette requête ;
- `false` : la limite a été atteinte entre la lecture du compteur et la tentative de réservation.

Une implémentation PostgreSQL ne doit jamais faire un simple `SELECT` puis `UPDATE` non protégé. La réservation doit utiliser une transaction, un verrou adapté ou une opération conditionnelle atomique.

## 4. Journalisation

`AccessDecisionJournal` sépare :

- `recordDecision` : toutes les décisions finales `ALLOW`, `DENY`, `REVIEW` ;
- `recordUsage` : uniquement les accès définitivement autorisés.

Chaque enregistrement doit conserver au minimum :

- `requestId` ;
- `correlationId` ;
- organisation ;
- site et terminal ;
- credential/droit lorsque connus ;
- décision et motif ;
- date métier ;
- moyen de paiement et montant approuvé lorsque présents.

La persistance devra rendre `requestId` idempotent afin qu’une répétition réseau ne crée pas deux usages ni deux consommations de quota.

## 5. Isolation multi-tenant

Le service refuse un `AccessTerminalProfile` dont `organizationId` ou `locationId` ne correspond pas à la requête.

Lorsque `request.terminalId` est fourni, le terminal chargé doit porter exactement le même identifiant.

Cette vérification s’ajoute aux protections déjà présentes dans le moteur de décision pour le credential et l’état de service. La future couche PostgreSQL devra également inclure `organizationId` dans ses clés de recherche et contraintes pertinentes.

## 6. Concurrence et quotas

La lecture initiale du nombre d’usages permet au moteur pur de refuser rapidement une limite déjà atteinte.

Elle ne suffit pas face à deux requêtes simultanées. Pour cette raison, après une première décision `ALLOW`, une réservation atomique est obligatoire lorsque `maxUsesPerPeriod` et `period` sont définis.

Si la réservation échoue, le service réévalue la même requête avec un compteur égal à la limite. La sortie reste donc produite par le moteur de décision commun et devient :

```text
DENY
USAGE_LIMIT_REACHED
```

Aucun usage autorisé n’est journalisé dans ce cas.

## 7. Profil terminal et bornes industrielles

Le profil terminal reste une donnée serveur décrivant les capacités réelles :

- hauteur simple ou double ;
- carte/NFC ;
- QR ;
- billets ;
- pièces ;
- rendu de monnaie ;
- imprimante ;
- interphone ;
- devises acceptées.

L’interface d’une borne ne doit pas afficher un moyen indisponible uniquement parce qu’il existe dans le produit global. Elle doit utiliser le croisement entre profil terminal et `AccessServiceAvailability`.

Le logiciel reste ainsi personnalisable pour un ministère, une société concessionnaire, un parking, une entreprise ou un opérateur de transport sans dupliquer le moteur métier.

## 8. Idempotence

La présente interface expose `requestId` et `correlationId` à tous les ports critiques. L’implémentation persistante suivante devra garantir :

1. une décision unique par `organizationId + requestId` ;
2. une réservation de quota unique par requête ;
3. un usage unique par requête ;
4. une reprise sûre après timeout entre réservation et réponse HTTP ;
5. aucune double action financière ou physique.

## 9. Tests automatisés

La suite :

`mansa-platform/packages/contracts/test/access-application-service.test.mjs`

couvre :

- orchestration d’un accès valide ;
- réservation de quota ;
- journalisation décision + usage ;
- échec concurrent de réservation converti en refus ;
- absence de réservation lorsqu’une décision est déjà refusée ;
- absence d’usage lors d’un refus ;
- rejet d’un profil terminal provenant d’un autre tenant ;
- rejet d’un terminal différent de celui demandé.

## 10. Étape suivante

La prochaine tranche doit implémenter les ports avec PostgreSQL/Prisma :

- tables ou modèles pour credentials, droits, états de service, terminaux, décisions, usages et réservations ;
- contraintes uniques d’idempotence ;
- transaction de réservation de quota ;
- tests d’intégration PostgreSQL avec deux requêtes concurrentes ;
- test de non-fuite multi-tenant ;
- service HTTP interne qui appelle `processAccessRequest` ;
- seulement ensuite, adaptateurs de paiement et contrôleur de voie/barrière.
