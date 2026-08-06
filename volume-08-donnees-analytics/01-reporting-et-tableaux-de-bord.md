# Volume 8 — Reporting et tableaux de bord

## 1. Objectif

Le domaine Analytics fournit des indicateurs agrégés, des tableaux de bord et des rapports exportables sans exposer directement les tables transactionnelles. Il sert les équipes opérations, finance, risque, conformité, partenaires et administration.

## 2. Principes

- Les données financières restent exprimées en unités mineures entières.
- Les agrégats ne remplacent jamais le grand livre comme source de vérité.
- Toute requête est limitée par le périmètre d’autorisation de l’acteur.
- Les dimensions sensibles sont minimisées, pseudonymisées ou masquées.
- Les rapports sont générés de manière asynchrone et possèdent une durée de conservation limitée.
- Les références de téléchargement sont temporaires et ne constituent pas des URL permanentes.
- Toute génération et tout téléchargement sont audités.

## 3. Contrats partagés

Le dépôt `mansa-platform` expose les modèles dans :

- `packages/contracts/src/analytics.ts` ;
- `packages/contracts/src/analytics-api.ts`.

Les routes initiales sont :

- `POST /v1/analytics/dashboard` ;
- `POST /v1/analytics/reports` ;
- `GET /v1/analytics/reports` ;
- `GET /v1/analytics/reports/:reportId` ;
- `GET /v1/analytics/reports/:reportId/download`.

## 4. Périodes et plages temporelles

Les périodes supportées sont `HOUR`, `DAY`, `WEEK`, `MONTH`, `QUARTER` et `YEAR`.

Une plage analytique contient :

- une date de début ISO 8601 ;
- une date de fin ISO 8601 ;
- un fuseau horaire explicite.

La date de début doit être antérieure ou égale à la date de fin. Le backend doit aussi appliquer une durée maximale selon le type de métrique, le rôle et la granularité demandée.

## 5. Métriques

Chaque série possède une clé stable, une unité et des points horodatés. Les unités initiales sont :

- `COUNT` ;
- `AMOUNT_MINOR` ;
- `PERCENTAGE` ;
- `DURATION_MS`.

Une série monétaire doit préciser sa devise. Les valeurs restent sérialisées sous forme de chaînes afin d’éviter toute perte de précision entre services et clients.

Exemples de métriques : volume et valeur des paiements, taux de succès, nouveaux utilisateurs, dossiers KYC, terminaux actifs, remboursements, litiges, revenus de commission, délais de traitement et écarts de rapprochement.

## 6. Rapports

Les formats initiaux sont `CSV`, `XLSX`, `PDF` et `JSON`. Le cycle de vie est :

`QUEUED` → `RUNNING` → `COMPLETED` ou `FAILED`, puis éventuellement `EXPIRED`.

Les statuts `COMPLETED`, `FAILED` et `EXPIRED` sont finaux. Une nouvelle tentative crée un nouveau rapport avec sa propre clé d’idempotence et sa propre trace d’audit.

## 7. Sécurité et confidentialité

- Aucun rapport ne doit contenir de PAN complet, CVV, code OTP, secret, mot de passe ou document KYC brut.
- Les exports de données personnelles nécessitent une permission dédiée et, selon le risque, une approbation à quatre yeux.
- Les filtres `countryCodes`, `merchantIds` et canaux sont intersectés avec le périmètre effectif de l’acteur.
- Les téléchargements doivent utiliser une référence signée à courte durée de vie.
- Les rapports arrivés à expiration sont supprimés du stockage objet selon la politique de rétention.
- Les journaux ne doivent pas enregistrer le contenu complet des rapports.

## 8. Traitement technique

1. L’API valide le contrat, l’autorisation, la plage et l’idempotence.
2. Une commande est publiée dans une file de génération.
3. Un worker lit une vue analytique ou un entrepôt, jamais les bases de production sans contrôle.
4. Le fichier est généré, contrôlé, chiffré au repos et stocké.
5. Le statut devient `COMPLETED` et une notification peut être émise.
6. Le téléchargement produit une référence temporaire et auditée.

## 9. Critères d’acceptation

- Une plage invalide est rejetée avant exécution.
- Une requête sans métrique est rejetée.
- Une unité ou période inconnue est rejetée.
- Une même clé d’idempotence ne crée pas deux rapports.
- Un utilisateur ne voit que les données de son périmètre.
- Une série monétaire conserve la précision exacte des unités mineures.
- Un rapport expiré ne peut plus être téléchargé.
- Les erreurs de génération produisent un code exploitable sans fuite de données.
- Les événements de création, fin, échec, expiration et téléchargement sont auditables.

## 10. Travaux backend restant à réaliser

- Persistance des demandes et statuts de rapports.
- Worker asynchrone et stockage objet.
- Catalogue versionné des métriques et rapports.
- Vues matérialisées ou entrepôt analytique.
- Contrôle RBAC/ABAC par dimension.
- Politiques de rétention et suppression.
- Tests de charge, précision financière, isolation multi-pays et sécurité des exports.
