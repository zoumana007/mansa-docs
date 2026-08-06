# Registre des modèles et décisions d’IA

## Objectif

Le registre de gouvernance recense chaque version de modèle utilisée par Mansa et permet de retrouver les décisions produites. Il fournit une source de vérité opérationnelle pour l’audit, la conformité, la sécurité, le support et les investigations.

## Contrats de référence

- `mansa-platform/packages/contracts/src/ai-governance.ts`
- `mansa-platform/packages/contracts/src/ai-governance-api.ts`

## Cas d’usage autorisés

Le socle initial distingue quatre cas d’usage :

- `JINI_ASSISTANT` ;
- `FRAUD_SCORING` ;
- `SUPPORT_TRIAGE` ;
- `RECOMMENDATION`.

Tout nouveau cas d’usage doit faire l’objet d’une analyse de risque, d’un propriétaire métier, d’un propriétaire technique, d’une politique de données et de critères de désactivation.

## Cycle de vie d’un modèle

Une version de modèle suit les statuts suivants :

1. `DRAFT` : version enregistrée mais non utilisée sur du trafic réel ;
2. `SHADOW` : exécution parallèle sans effet sur la décision finale ;
3. `ACTIVE` : version autorisée à participer au parcours métier ;
4. `SUSPENDED` : utilisation interrompue immédiatement ;
5. `RETIRED` : version retirée définitivement du service actif.

Un changement de statut est idempotent, justifié et audité. L’activation en production doit pouvoir exiger une demande d’approbation selon le niveau de risque du cas d’usage.

## Données du registre

Chaque version conserve au minimum :

- identifiant stable du modèle ;
- version immuable ;
- cas d’usage ;
- fournisseur ou moteur ;
- classification des données ;
- statut courant ;
- date de déploiement ou de retrait ;
- références d’évaluation et d’approbation dans les systèmes internes.

Aucune clé d’API, aucun secret fournisseur, aucun prompt contenant des données personnelles réelles et aucun jeu de données brut ne doit être enregistré dans GitHub.

## Traces de décision

Chaque décision conserve :

- un identifiant unique ;
- le cas d’usage ;
- le modèle et sa version ;
- le type et l’identifiant du sujet ;
- le résultat ;
- le score éventuel ;
- les codes de raison ;
- l’identifiant de corrélation ;
- l’horodatage ;
- l’indication d’une revue humaine obligatoire.

Les codes de raison doivent être stables, documentés et exploitables sans exposer une logique sensible permettant de contourner les contrôles antifraude.

## API de gouvernance

Le contrat partagé expose :

- `GET /v1/ai/models` pour filtrer les versions par cas d’usage, statut ou fournisseur ;
- `POST /v1/ai/models` pour enregistrer une version ;
- `GET /v1/ai/models/:modelId/versions/:version` pour consulter une version ;
- `POST /v1/ai/models/:modelId/versions/:version/status` pour modifier son statut ;
- `GET /v1/ai/decisions` pour rechercher les décisions ;
- `GET /v1/ai/decisions/:decisionId` pour consulter une trace précise.

Les listes sont paginées par curseur. Les filtres temporels utilisent des dates ISO 8601 et sont validés côté API.

## Autorisations minimales

- La lecture du registre est réservée aux rôles IA, risque, sécurité, conformité et audit autorisés.
- L’enregistrement d’une version est séparé de son activation.
- La suspension d’urgence peut être accordée à un rôle restreint et produit toujours un événement d’audit.
- Les traces de décision client ne sont accessibles qu’avec un besoin métier démontré et un périmètre de données limité.
- Les exports sont chiffrés, temporaires et journalisés.

## Résilience

Le registre n’est pas dans le chemin critique d’un paiement déjà autorisé. Les services consommateurs utilisent une configuration versionnée et mise en cache, avec une date d’expiration courte et un mécanisme de révocation prioritaire pour les suspensions.

Une décision tardive ou rejouée ne peut jamais modifier rétroactivement une transaction finalisée. Toute correction passe par les procédures métier et comptables prévues.

## Critères d’acceptation

1. Une version est identifiable par le couple `modelId` et `version`.
2. Les statuts et cas d’usage inconnus sont rejetés.
3. Un changement de statut exige une raison non vide et une clé d’idempotence.
4. Une décision est recherchable par corrélation, sujet, modèle, cas d’usage et période.
5. Les résultats nécessitant une revue humaine sont filtrables.
6. Les endpoints ne renvoient aucun secret ni donnée personnelle non nécessaire.
7. La documentation des routes correspond au contrat TypeScript publié.
8. La suspension d’une version active est visible dans l’audit et propagée aux consommateurs.
