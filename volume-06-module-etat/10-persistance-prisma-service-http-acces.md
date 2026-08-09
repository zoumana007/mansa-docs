# Persistance Prisma et service interne — accès et mobilité

## 1. Objet

Cette tranche relie le moteur déterministe d’accès aux tables PostgreSQL introduites dans la tranche précédente et à l’API Gateway interne.

Elle matérialise dans `mansa-platform` :

- `PrismaAccessRepository` ;
- la réservation atomique de quota ;
- le journal idempotent des décisions et usages ;
- la relecture d’une décision déjà enregistrée ;
- `AccessService` ;
- le contrôleur interne d’évaluation ;
- `AccessModule` dans l’API Gateway.

## 2. Repository Prisma

Le fichier :

`apps/api-gateway/src/access/access.repository.ts`

implémente simultanément les trois ports applicatifs :

- `AccessApplicationRepository` ;
- `AccessQuotaReservation` ;
- `AccessDecisionJournal`.

Le repository conserve explicitement `organizationId` dans toutes les lectures et écritures où l’identité métier pourrait se répéter entre exploitants.

## 3. Résolution des credentials et droits

La résolution d’un credential se fait par :

```text
organizationId + credentialType + publicReference
```

Le droit est ensuite recherché dans le même tenant avec :

```text
organizationId + subjectId + useCase
```

Le moteur ne reçoit donc pas un credential ou un entitlement provenant d’une autre organisation.

## 4. Disponibilité et profil matériel

La disponibilité de service est chargée dans le périmètre :

```text
organizationId + locationId + laneKey
```

Lorsqu’un `terminalId` est fourni, son profil sert à retrouver la voie correspondante.

Le profil retourné expose les capacités physiques déjà prévues dans le contrat :

- hauteur simple ou double ;
- carte/NFC ;
- QR ;
- Mobile Money ;
- billets et pièces ;
- coupures acceptées ;
- rendu de monnaie ;
- imprimante ;
- interphone.

## 5. Fenêtres de quota

Les périodes `DAY`, `WEEK` et `MONTH` utilisent le helper partagé :

`calculateAccessQuotaWindow()`.

Les fenêtres sont calculées en UTC afin que l’API, les workers et PostgreSQL utilisent exactement les mêmes bornes temporelles.

## 6. Réservation atomique

La réservation de quota utilise une transaction PostgreSQL `SERIALIZABLE`.

Le flux est :

1. rechercher une réservation existante pour la même requête ;
2. créer ou mettre à jour le compteur de période ;
3. incrémenter uniquement si `used < limit` ;
4. créer la réservation idempotente dans la même transaction ;
5. valider la transaction ;
6. en cas de collision unique, relire la réservation et traiter le replay comme un succès ;
7. en cas de conflit de sérialisation, rejouer l’opération.

Le couple compteur + réservation ne peut donc pas être validé à moitié.

## 7. Journal multi-tenant

Le contrat `AccessDecisionJournal` reçoit désormais la requête d’accès en plus de la décision ou de l’usage.

Cette évolution est nécessaire car une décision de refus peut être produite avant qu’un credential ou un entitlement ne soit trouvé. Le journal ne doit jamais essayer de déduire le tenant depuis une ressource qui peut légitimement être absente.

Les signatures sont donc :

```text
recordDecision(request, decision)
recordUsage(request, usage)
```

`organizationId` vient toujours de la requête d’origine.

## 8. Idempotence d’évaluation

`AccessService` vérifie d’abord :

```text
organizationId + requestId
```

Si une décision est déjà enregistrée, elle est retournée directement sans :

- recalculer le moteur ;
- réserver une seconde fois le quota ;
- créer un second usage.

Cette règle protège notamment les retries réseau des bornes et contrôleurs de voie.

## 9. API HTTP interne

Le module expose :

```http
POST /v1/internal/access/evaluate
```

Le contrôleur est protégé par `InternalServiceGuard`, comme les routes internes de rapprochement.

Le corps correspond au contrat `AccessRequest` :

```json
{
  "requestId": "req-42",
  "organizationId": "org-1",
  "useCase": "TOLL",
  "credentialType": "RFID_UHF_TAG",
  "credentialReference": "tag-1",
  "observedLicensePlate": "AA-123-AA",
  "locationId": "toll-1",
  "terminalId": "lane-1",
  "paymentMethod": "SUBSCRIPTION",
  "occurredAt": "2026-08-09T15:00:00.000Z",
  "correlationId": "corr-42"
}
```

Le résultat est un `AccessDecision` déterministe.

## 10. Sécurité et isolation

Cette tranche conserve quatre règles :

- aucune décision n’est enregistrée sans le tenant de la requête ;
- les retries d’un tenant ne peuvent pas relire la décision d’un autre tenant ;
- le profil de terminal doit correspondre à l’organisation et au lieu demandés ;
- la route HTTP n’est pas publique et nécessite l’authentification service-à-service interne.

## 11. Validation déjà couverte

Les tests de contrat du service applicatif vérifient désormais que la requête complète, y compris son `organizationId`, est transmise au journal de décision et d’usage.

Les tests existants continuent à couvrir :

- décision autorisée ;
- quota atteint après collision ;
- service fermé ;
- terminal d’un autre tenant ;
- terminal différent de celui demandé.

## 12. Prochaine tranche

Le socle est branché, mais la persistance d’accès n’est pas encore considérée comme terminée tant que les validations PostgreSQL réelles suivantes ne sont pas présentes dans la CI :

- deux tenants utilisant le même `requestId` sans collision ;
- relecture idempotente d’une décision ;
- deux requêtes concurrentes visant la dernière unité d’un quota ;
- replay concurrent de la même réservation ;
- rollback vérifié lorsque la création de réservation échoue ;
- test HTTP du contrôleur interne avec garde active.

La tranche suivante doit ajouter ces tests d’intégration PostgreSQL et leur job CI avant d’élargir l’API publique de gestion des credentials, entitlements et états de borne.
