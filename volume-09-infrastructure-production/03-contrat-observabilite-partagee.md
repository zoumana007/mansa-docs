# Volume 9 — Contrat d’observabilité partagé

## 1. Objectif

Le package `mansa-platform/packages/observability` fournit les primitives communes que les applications, services backend et workers utilisent pour produire des journaux structurés, propager la corrélation, exposer leur état de santé et qualifier les incidents.

Ce package ne remplace ni le journal d’audit métier ni le grand livre financier. Il fournit uniquement les conventions techniques communes nécessaires à l’exploitation.

## 2. Corrélation de bout en bout

Chaque requête ou traitement asynchrone doit transporter au minimum un `correlationId`. Lorsque le contexte existe, il peut également contenir un `requestId`, un `traceId`, un `spanId`, un identifiant d’acteur, un identifiant de session et le code pays.

Le même identifiant de corrélation doit être conservé lors des passages suivants :

- API Gateway vers module métier ;
- module métier vers adaptateur partenaire ;
- production puis consommation d’un événement ;
- traitement worker ;
- notification déclenchée par une opération ;
- incident ou erreur technique relié à une transaction.

## 3. Journaux structurés

Les niveaux communs sont `TRACE`, `DEBUG`, `INFO`, `WARN`, `ERROR` et `FATAL`.

Un événement de journal comporte au minimum :

- un horodatage ISO ;
- le niveau ;
- le nom du service ;
- un nom d’événement stable ;
- un message exploitable ;
- le contexte de corrélation.

Les attributs complémentaires doivent être limités aux informations utiles au diagnostic. Les clés contenant des secrets ou données d’authentification doivent être masquées avant écriture.

Sont notamment interdits dans les journaux : mots de passe, secrets, jetons, codes OTP, PIN, CVV/CVC, numéro de carte complet et clé privée.

Le masquage doit s’appliquer récursivement aux objets et tableaux imbriqués. Une référence circulaire rencontrée dans les attributs techniques doit être neutralisée avant sérialisation afin d’éviter une fuite ou un échec de journalisation. La fonction commune `sanitizeStructuredLogEvent` constitue le point de passage recommandé avant émission d’un événement vers un fournisseur de logs.

## 4. Santé des services

Les états partagés sont :

- `HEALTHY` : service et dépendances nécessaires disponibles ;
- `DEGRADED` : une ou plusieurs dépendances sont dégradées sans interruption complète ;
- `UNHEALTHY` : le service ne peut plus assurer son rôle critique.

Une sonde de santé ne doit jamais révéler un secret, une chaîne de connexion ou une donnée personnelle. Les dépendances peuvent exposer un code de diagnostic non sensible et une latence.

## 5. Incidents

Les niveaux d’incident sont alignés sur le référentiel du volume 9 : `P1`, `P2`, `P3`, `P4`.

Un enregistrement d’incident doit conserver : identifiant, gravité, titre, dates de début/détection/résolution, services affectés et, lorsque disponible, les identifiants de corrélation nécessaires au diagnostic.

Les incidents `P1` et `P2` doivent être reliés à un runbook et faire l’objet d’une vérification financière lorsque des paiements, soldes, règlements ou rapprochements ont pu être affectés.

## 6. Métriques

Les définitions de métriques partagées précisent : nom stable, description, unité et labels autorisés.

Les unités communes du socle sont :

- `COUNT` ;
- `MILLISECONDS` ;
- `BYTES` ;
- `RATIO` ;
- `AMOUNT_MINOR`.

Pour `AMOUNT_MINOR`, la devise doit être portée par un label autorisé ou par le contexte métier correspondant. Aucun montant financier ne doit être converti en flottant pour la télémétrie métier.

## 7. Cardinalité et données sensibles

Les métriques ne doivent pas utiliser comme labels des identifiants à cardinalité non bornée tels que `userId`, `transactionId`, `requestId` ou `correlationId`.

Les identifiants de corrélation appartiennent aux journaux et traces, pas aux labels de métriques.

Les données KYC, numéros de téléphone complets, adresses e-mail complètes et données de carte ne doivent pas être introduits dans les métriques.

## 8. Implémentation de référence

Le dépôt plateforme contient désormais :

- `packages/observability/package.json` ;
- `packages/observability/tsconfig.json` ;
- `packages/observability/src/index.ts` ;
- `packages/observability/test/observability.test.mjs`.

Le module fournit les types de corrélation, journaux, santé, dépendances, incidents et métriques ainsi que des fonctions communes de validation, classification de santé, masquage récursif des attributs sensibles et préparation d’un événement de journal assaini.

La suite de tests construit le package puis vérifie le masquage des secrets imbriqués, la neutralisation des références circulaires, la validation des événements structurés et la classification déterministe de l’état de santé.

## 9. Critères d’acceptation

- Le package compile en TypeScript strict.
- Tous les services peuvent importer les primitives sans dépendre d’un fournisseur d’observabilité particulier.
- Les niveaux d’incident restent identiques entre documentation et code.
- Une clé sensible reconnue est remplacée par `[REDACTED]` avant journalisation, y compris lorsqu’elle est imbriquée dans un objet ou un tableau.
- Une structure circulaire est neutralisée et ne provoque pas de récursion infinie pendant le masquage.
- Un événement sans `correlationId` valide est rejeté par la validation commune.
- La classification de santé produit `HEALTHY`, `DEGRADED` ou `UNHEALTHY` de manière déterministe.
- Les tests du package sont exécutés via le script `test` après compilation TypeScript.
- Aucun secret ou identifiant de production n’est présent dans le dépôt.

## 10. Étapes suivantes

Les prochaines couches doivent brancher ces primitives sur les implémentations réelles de logs, métriques et traces, puis définir les tableaux de bord, alertes et runbooks de chaque parcours critique. Le choix du fournisseur d’observabilité reste un détail d’infrastructure et ne doit pas contaminer les contrats métier partagés.
