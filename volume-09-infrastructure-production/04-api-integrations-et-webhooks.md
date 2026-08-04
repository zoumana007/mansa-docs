# API d’intégrations partenaires et webhooks

## 1. Objet

Ce catalogue définit les contrats transversaux utilisés pour enregistrer les partenaires externes, piloter leur statut, gérer la rotation de leurs moyens d’authentification et traiter les webhooks entrants ou sortants.

La référence technique est `packages/contracts/src/integration-api.ts` dans `mansa-platform`. Le catalogue est exporté par le registre `@mansa/contracts/api-contracts`.

## 2. Périmètre

Les partenaires couverts sont notamment :

- banques partenaires ;
- opérateurs Mobile Money ;
- processeurs et réseaux de cartes ;
- administrations et services publics ;
- fournisseurs d’identité ;
- fournisseurs de notifications ;
- autres prestataires techniques explicitement approuvés.

Aucun secret, certificat, clé privée ou jeton n’est retourné par ces contrats.

## 3. Préfixe et routes

Préfixe : `/v1/integrations`.

| Opération | Méthode | Route | Permission minimale |
| --- | --- | --- | --- |
| Lister les partenaires | `GET` | `/partners` | `integration.partner.read` |
| Lire un partenaire | `GET` | `/partners/:partnerId` | `integration.partner.read` |
| Enregistrer un partenaire | `POST` | `/partners` | `integration.partner.create` |
| Modifier son statut | `POST` | `/partners/:partnerId/status` | `integration.partner.status.update` |
| Planifier une rotation d’identifiants | `POST` | `/partners/:partnerId/credentials/rotation` | `integration.credential.rotate` |
| Recevoir un webhook | `POST` | `/webhooks/:partnerId/:eventType` | authentification partenaire |
| Rejouer une livraison | `POST` | `/webhook-deliveries/:deliveryId/replay` | `integration.webhook.replay` |
| Lire une livraison | `GET` | `/webhook-deliveries/:deliveryId` | `integration.webhook.read` |

## 4. Cycle de vie d’un partenaire

Les statuts sont :

- `DRAFT` : configuration non activée ;
- `ACTIVE` : trafic autorisé ;
- `DEGRADED` : trafic autorisé avec surveillance renforcée ;
- `SUSPENDED` : trafic bloqué temporairement ;
- `DISABLED` : intégration désactivée.

Toute transition exige un motif, une clé d’idempotence et un événement d’audit. Une suspension doit interrompre immédiatement les nouvelles opérations, sans supprimer les données de traçabilité.

## 5. Gestion des secrets et identifiants

Les types de rotation couverts sont : clé API, client OAuth, clé de signature et certificat mTLS.

Les règles obligatoires sont les suivantes :

1. Les secrets sont stockés exclusivement dans un gestionnaire de secrets.
2. Les réponses API ne contiennent jamais la valeur du secret.
3. Une rotation produit un identifiant de suivi et un statut.
4. L’ancien et le nouveau moyen d’authentification peuvent coexister pendant une courte fenêtre contrôlée.
5. Toute rotation privilégiée nécessite une authentification forte et, en production, une double approbation.
6. Les dates d’effet et d’expiration sont explicites et auditables.

## 6. Webhooks

Chaque webhook transporte au minimum :

- `deliveryId` ;
- `eventId` ;
- `eventType` ;
- `occurredAt` ;
- `correlationId` ;
- `payload`.

Le serveur vérifie avant traitement :

- l’identité du partenaire ;
- la signature ;
- l’horodatage et la fenêtre anti-rejeu ;
- l’unicité de `deliveryId` et `eventId` ;
- la taille et le schéma du message ;
- le statut actif du partenaire.

Une livraison déjà acceptée retourne `DUPLICATE` sans rejouer l’effet métier. Les traitements longs sont placés en file et accusés réception rapidement.

## 7. Rejeu, erreurs et file morte

Les statuts de livraison sont `PENDING`, `DELIVERED`, `FAILED` et `DEAD_LETTER`.

Les tentatives utilisent une temporisation exponentielle bornée avec aléa. Les erreurs fonctionnelles définitives ne sont pas retentées automatiquement. Une livraison en file morte peut être rejouée uniquement après correction de la cause, avec motif et clé d’idempotence.

Les réponses et journaux ne doivent pas exposer de corps contenant des données personnelles, de PAN complet, de document KYC ou de secret partenaire.

## 8. Observabilité

Les métriques minimales sont :

- taux de succès par partenaire et type d’événement ;
- latence de réception et de traitement ;
- nombre de tentatives ;
- profondeur des files ;
- volume en file morte ;
- erreurs de signature, schéma et authentification ;
- durée depuis la dernière livraison réussie.

Les alertes sont différenciées par criticité et environnement. Une dégradation partenaire ne doit pas provoquer automatiquement une panne générale de la plateforme.

## 9. Critères de recette

1. Un partenaire suspendu ne peut plus envoyer ou recevoir de nouvelle opération.
2. Un webhook dont la signature est invalide est refusé sans traitement métier.
3. Un même `deliveryId` ne produit qu’un seul effet métier.
4. Une rotation ne révèle jamais la valeur du nouvel identifiant.
5. Une mutation rejouée avec la même clé d’idempotence ne crée pas de doublon.
6. Une erreur temporaire déclenche des tentatives bornées.
7. Une erreur définitive rejoint la file morte avec un diagnostic exploitable.
8. Un rejeu manuel est autorisé, justifié et audité.
9. Les routes, méthodes, statuts et types correspondent à `integration-api.ts`.
10. Aucun secret ou donnée sensible brute n’est présent dans le dépôt, les réponses ou les journaux.
