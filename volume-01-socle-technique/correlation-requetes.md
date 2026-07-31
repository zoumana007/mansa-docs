# Corrélation des requêtes API

## Objectif

Chaque requête HTTP traversant l’API Gateway Mansa doit posséder un identifiant de corrélation stable afin de relier les journaux, erreurs, audits, métriques et appels vers les services internes.

## Contrat HTTP

- En-tête entrant et sortant : `X-Correlation-Id`.
- Si le client fournit une valeur non vide de 128 caractères maximum, l’API la conserve.
- Si l’en-tête est absent, vide, multiple sans première valeur exploitable ou trop long, l’API génère un UUID.
- L’identifiant retenu est renvoyé dans la réponse HTTP.
- L’identifiant est attaché à l’objet requête pour être réutilisé par les journaux et futurs adaptateurs.

## Implémentation initiale

Le portail API utilise un intercepteur NestJS global :

```text
apps/api-gateway/src/correlation.interceptor.ts
```

Il est enregistré avec `APP_INTERCEPTOR` dans :

```text
apps/api-gateway/src/app.module.ts
```

## Règles de sécurité

- Ne jamais considérer l’identifiant fourni par le client comme une preuve d’identité.
- Ne jamais y encoder de donnée personnelle, secret, jeton ou référence bancaire.
- Limiter sa longueur avant journalisation.
- Échapper ou sérialiser les journaux structurés afin d’éviter l’injection de lignes de logs.

## Évolutions prévues

1. Propager l’identifiant dans les appels HTTP, messages asynchrones et événements de domaine.
2. L’intégrer au logger structuré et aux traces OpenTelemetry.
3. L’inclure dans le format standard des erreurs API.
4. Ajouter des tests d’intégration HTTP couvrant conservation, génération et rejet des valeurs trop longues.

## Critères d’acceptation

- Une requête sans en-tête reçoit un `X-Correlation-Id` non vide dans la réponse.
- Une requête avec un identifiant valide récupère exactement le même identifiant.
- Une valeur de plus de 128 caractères est remplacée par un identifiant généré.
- Toutes les routes, y compris `/api/v1/health`, appliquent le comportement.
- Le mécanisme ne modifie ni le corps ni le statut de la réponse.

## Hypothèse à valider

Le format UUID est retenu pour le socle. Le choix définitif entre UUIDv4, UUIDv7 ou identifiant compatible avec l’infrastructure de traçage sera confirmé lors de l’intégration OpenTelemetry.
