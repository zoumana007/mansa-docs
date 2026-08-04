# Catalogue API — Analytics, tableaux de bord et rapports

## 1. Objet

Ce catalogue décrit les contrats HTTP du premier socle analytics de Mansa. Il couvre la consultation de séries de métriques, la génération asynchrone de rapports, le suivi de leur état et l’obtention d’une référence de téléchargement temporaire.

La source TypeScript correspondante est `packages/contracts/src/analytics-api.ts` dans `mansa-platform`.

## 2. Principes communs

- Toutes les routes sont versionnées sous `/v1`.
- Les montants sont exprimés en unités mineures et sérialisés sous forme de chaînes lorsque nécessaire.
- Les dates utilisent ISO 8601 et chaque plage précise son fuseau horaire.
- Les résultats sont filtrés selon le pays, l’organisation, le rôle et le périmètre de l’acteur.
- La génération d’un rapport est idempotente.
- Les références de téléchargement sont temporaires, révocables et ne contiennent aucun secret durable.
- Les réponses d’erreur suivent le contrat commun `ApiErrorResponse`.

## 3. Tableau de bord

### `POST /v1/analytics/dashboard`

Retourne un instantané agrégé correspondant à `DashboardQuery`.

La requête contient :

- une plage `from`, `to` et `timezone` ;
- une liste de clés de métriques ;
- une période d’agrégation ;
- des filtres facultatifs par pays, commerçant et canal.

La réponse `DashboardSnapshot` contient la date de génération, la plage demandée et les séries de métriques. Une série indique son unité, sa devise éventuelle et ses points ordonnés.

Contraintes :

- aucune série ne doit révéler une donnée personnelle directement identifiable ;
- les filtres hors périmètre sont refusés plutôt qu’ignorés silencieusement ;
- les données financières doivent être réconciliables avec les sources de référence ;
- les résultats peuvent être mis en cache selon une durée explicitement documentée.

## 4. Rapports

### `POST /v1/analytics/reports`

Crée une demande de rapport à partir de `CreateReportCommand`.

Formats initiaux : `CSV`, `XLSX`, `PDF` et `JSON`.

La réponse `GeneratedReport` est créée avec un état initial `QUEUED` ou `RUNNING`. Une nouvelle requête avec la même clé d’idempotence ne doit pas créer un second rapport logique.

### `GET /v1/analytics/reports/:reportId`

Retourne l’état courant d’un rapport : `QUEUED`, `RUNNING`, `COMPLETED`, `FAILED` ou `EXPIRED`.

Les codes d’échec sont stables, documentés et ne doivent pas exposer d’informations techniques sensibles.

### `GET /v1/analytics/reports`

Retourne une page de rapports autorisés. Les filtres disponibles sont : état, type de rapport, demandeur et plage de création.

La pagination suit `PageRequest` et la réponse suit `PageResponse<GeneratedReport>`.

### `GET /v1/analytics/reports/:reportId/download`

Retourne `ReportDownloadReference` uniquement lorsque le rapport est terminé et encore disponible.

La référence doit :

- expirer rapidement ;
- être liée au rapport et à l’acteur autorisé ;
- être renouvelable sans régénérer le rapport ;
- être inutilisable après expiration ou révocation.

## 5. Autorisations minimales

- Client : uniquement ses relevés et rapports personnels explicitement disponibles.
- Commerçant : tableaux de bord et rapports de son organisation.
- Finance : rapports financiers et de rapprochement selon son périmètre.
- Risque/Fraude : métriques et rapports de risque autorisés.
- Administration publique : rapports du service public et de l’organisation concernés.
- Super administrateur : accès contrôlé et audité, sans contournement automatique de la confidentialité.
- Service technique : identité de service dédiée, permissions minimales et finalité déclarée.

## 6. Audit, conservation et observabilité

Chaque demande conserve : identifiant de corrélation, acteur, périmètre, filtres, type de rapport, format, statut, durée, volume produit et référence d’audit.

Les journaux ne doivent pas contenir le contenu du rapport ni des données personnelles en clair.

Les métriques minimales sont : temps de réponse du tableau de bord, taux de cache, rapports en attente, durée de génération, taux d’échec, taille des rapports, expirations et téléchargements refusés.

La durée de conservation des rapports doit être configurable par type, pays et obligation réglementaire.

## 7. Comportement en cas d’indisponibilité

- Le tableau de bord peut retourner une erreur temporaire ou un instantané explicitement marqué comme ancien.
- Une demande de rapport acceptée reste récupérable après reprise grâce à une file persistante.
- Une reprise ne doit jamais produire plusieurs rapports pour la même clé d’idempotence.
- Une référence de téléchargement ne doit pas être créée avant la disponibilité complète du fichier.

## 8. Critères d’acceptation

1. Les cinq routes et méthodes correspondent exactement au contrat TypeScript.
2. Les plages de dates et fuseaux sont validés.
3. Les filtres respectent strictement le périmètre d’autorisation.
4. Les créations de rapports sont idempotentes.
5. Les états suivent le cycle prévu et les rapports expirés ne sont plus téléchargeables.
6. Les références de téléchargement sont temporaires et révocables.
7. Les métriques financières peuvent être rapprochées des sources métier.
8. Aucun secret ni contenu personnel non nécessaire n’apparaît dans les journaux.
