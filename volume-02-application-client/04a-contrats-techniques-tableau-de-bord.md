# Volume 02 — Application Client

# Complément 04A — Contrats techniques du tableau de bord

## 1. Objet

Ce complément traduit le chapitre 04 en contrats TypeScript partagés entre l'application Client, l'API Gateway et les futurs modules backend.

La source de vérité technique se trouve dans :

```text
mansa-platform/packages/contracts/src/dashboard.ts
```

Le sous-chemin public du paquet est :

```text
@mansa/contracts/dashboard
```

## 2. Catalogues normalisés

Les valeurs suivantes sont fermées et validées par des gardes de type :

- `DASHBOARD_WIDGET_TYPES` : catégories de widgets disponibles ;
- `DASHBOARD_WIDGET_SIZES` : tailles `COMPACT`, `STANDARD` et `LARGE` ;
- `DASHBOARD_ALERT_SEVERITIES` : niveaux `INFO`, `WARNING` et `CRITICAL` ;
- `QUICK_ACTION_TYPES` : actions rapides reconnues par le client.

Une nouvelle valeur ne doit pas être utilisée dans une application avant d'avoir été ajoutée au contrat, documentée et couverte par un test.

## 3. Modèles partagés

Le contrat expose :

- `DashboardAccountSummary` ;
- `DashboardCardSummary` ;
- `DashboardActivityItem` ;
- `DashboardAlert` ;
- `QuickAction` ;
- `DashboardWidget` ;
- `DashboardLayout` ;
- `DashboardSnapshot` ;
- `UpdateDashboardLayoutCommand`.

Les montants réutilisent obligatoirement le type `Money` du socle afin d'éviter les nombres flottants.

## 4. Versionnement de la disposition

`DashboardLayout.version` permet un contrôle de concurrence optimiste.

Lors d'une mise à jour, le client envoie `expectedVersion`. L'API refuse la modification lorsque la version courante a changé depuis la dernière lecture. Le client recharge alors la disposition avant de proposer une nouvelle modification.

Cette règle évite qu'un second téléphone écrase silencieusement la personnalisation effectuée sur le premier.

## 5. Widgets administrés

Un widget marqué `mandatory: true` :

- reste visible pour les utilisateurs ciblés ;
- ne peut pas être supprimé localement ;
- peut néanmoins changer de taille ou de position si la politique administrative l'autorise ;
- doit être désactivé par configuration centrale et non par suppression de données.

`countryCodes` limite l'exposition d'un widget aux pays autorisés.

## 6. Données sensibles

Le tableau de bord ne transporte jamais :

- le numéro complet d'une carte ;
- le cryptogramme ;
- le code PIN ;
- un secret d'authentification ;
- une pièce KYC brute.

`DashboardCardSummary.lastFour` contient uniquement les quatre derniers chiffres nécessaires à l'affichage.

## 7. API cible

Les routes du chapitre 04 pourront retourner les contrats suivants :

| Route | Contrat principal |
|---|---|
| `GET /dashboard` | `DashboardSnapshot` |
| `GET /dashboard/widgets` | `DashboardWidget[]` |
| `PATCH /dashboard/layout` | `UpdateDashboardLayoutCommand` |
| `GET /dashboard/activity` | `DashboardActivityItem[]` |
| `GET /dashboard/cards` | `DashboardCardSummary[]` |
| `GET /dashboard/accounts` | `DashboardAccountSummary[]` |
| `GET /dashboard/alerts` | `DashboardAlert[]` |

Les réponses HTTP définitives devront être enveloppées avec les conventions communes de pagination, corrélation et erreurs API lorsque nécessaire.

## 8. Validation

Le paquet de contrats contient des tests vérifiant :

- l'acceptation de chaque valeur déclarée ;
- le rejet de valeurs inconnues ;
- l'absence de doublons dans les catalogues ;
- la disponibilité du sous-chemin `@mansa/contracts/dashboard` après compilation.

## 9. Évolutions prévues

Les lots suivants devront compléter ce socle avec :

- schémas de validation à l'entrée de l'API ;
- module backend d'agrégation du tableau de bord ;
- cache utilisateur à durée courte ;
- synchronisation temps réel des alertes et activités ;
- persistance versionnée des préférences ;
- télémétrie sans données financières sensibles.
