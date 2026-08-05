# Abonnements et paiements récurrents

## Objectif

Le module d’abonnements aide le client à identifier, suivre et comprendre les paiements récurrents débités depuis ses portefeuilles. Il ne résilie pas automatiquement un contrat chez un fournisseur et ne bloque pas un paiement sans action explicite prise dans un module de contrôle autorisé.

## Sources

Un abonnement possède l’une des sources suivantes :

- `AUTOMATIC` : détecté à partir de transactions récurrentes ;
- `MANUAL` : ajouté par le client.

Une détection automatique reste une hypothèse tant que le client ne l’a pas confirmée. Le score de confiance est compris entre 0 et 1 et ne doit jamais être présenté comme une certitude.

## Fréquences

- `WEEKLY` : prélèvement attendu chaque semaine ;
- `MONTHLY` : prélèvement attendu chaque mois ;
- `QUARTERLY` : prélèvement attendu chaque trimestre ;
- `YEARLY` : prélèvement attendu chaque année ;
- `IRREGULAR` : récurrence reconnue sans calendrier assez stable.

Les dates attendues sont indicatives. Les jours non ouvrés, délais de compensation et variations de calendrier peuvent décaler un débit réel.

## Statuts

- `DETECTED` : candidat détecté automatiquement, à confirmer ou ignorer ;
- `CONFIRMED` : abonnement reconnu par le client ;
- `IGNORED` : candidat masqué des vues principales sans suppression de l’historique ;
- `CANCELLED` : abonnement déclaré terminé par le client.

Le statut `CANCELLED` décrit le suivi dans Mansa. Il ne prouve pas qu’un contrat externe est juridiquement résilié.

## Détection

La détection peut s’appuyer sur le commerçant, le libellé normalisé, les montants, les intervalles entre opérations et l’historique du portefeuille. Elle respecte les règles suivantes :

1. aucune transaction n’est modifiée par le moteur de détection ;
2. un candidat doit référencer au moins une transaction réelle ;
3. les données d’un client ne sont pas utilisées pour exposer les habitudes d’un autre client ;
4. les faux positifs peuvent être ignorés et servent uniquement à améliorer les modèles selon les consentements applicables ;
5. une variation de montant n’empêche pas automatiquement la reconnaissance d’un abonnement ;
6. chaque recalcul conserve une trace de version et de justification technique.

## Parcours client

### Liste

L’application affiche les abonnements confirmés, les candidats à examiner, le montant attendu, la fréquence, le dernier débit et la prochaine date estimée. Les montants sont présentés dans leur devise réelle et ne sont jamais additionnés sans conversion explicite.

### Confirmation

Le client peut confirmer un candidat, corriger son nom, sa catégorie, son montant attendu ou sa fréquence. La confirmation ne modifie pas les transactions historiques.

### Création manuelle

Le client choisit un portefeuille, un commerçant ou un libellé, un montant attendu, une fréquence et éventuellement une prochaine date. Un abonnement manuel ne génère aucun débit automatique.

### Annulation du suivi

Le client peut marquer l’abonnement comme annulé ou ignorer un candidat. L’application rappelle clairement qu’il doit contacter le fournisseur lorsque la résiliation du contrat est nécessaire.

## Alertes

Le service peut notifier :

- l’arrivée prochaine d’un débit attendu ;
- un montant sensiblement supérieur au montant habituel ;
- un nouveau candidat détecté ;
- la reprise d’un abonnement précédemment annulé ;
- plusieurs débits rapprochés potentiellement anormaux.

Les alertes sont idempotentes, respectent les préférences de notification et ne révèlent pas de détails sensibles sur un écran verrouillé.

## API alignée avec le dépôt plateforme

Contrats maintenus dans :

- `packages/contracts/src/subscription.ts` ;
- `packages/contracts/src/subscription-api.ts`.

Exports publics :

- `@mansa/contracts/subscription` ;
- `@mansa/contracts/subscription-api`.

Routes principales :

- `GET /v1/subscriptions` ;
- `POST /v1/subscriptions` ;
- `GET /v1/subscriptions/:subscriptionId` ;
- `PATCH /v1/subscriptions/:subscriptionId` ;
- `GET /v1/subscriptions/:subscriptionId/charges`.

Les listes sont paginées et filtrables par propriétaire, portefeuille, statut, fréquence et source.

## Sécurité et confidentialité

- Un client ne consulte et ne modifie que ses propres abonnements.
- Toute consultation administrative sensible exige une permission et un motif audité.
- Les libellés sont nettoyés pour éviter l’exposition d’un numéro de carte, d’un compte ou d’un secret.
- Les références de transactions sont conservées sans dupliquer les données sensibles.
- Les mécanismes d’intelligence artificielle appliquent les règles de consentement, minimisation et gouvernance définies au volume 7.

## Critères de recette

1. Un score de confiance inférieur à 0 ou supérieur à 1 est rejeté.
2. Un candidat automatique exige une transaction source et une date de dernier débit.
3. Un abonnement manuel ne déclenche jamais de paiement.
4. Un candidat ignoré disparaît des vues principales mais reste auditable.
5. La confirmation d’un candidat ne modifie aucune transaction historique.
6. Les montants de devises différentes ne sont pas agrégés sans conversion explicite.
7. Une alerte de prochain débit n’est envoyée qu’une fois pour la même échéance et le même canal.
8. Le statut annulé n’est jamais présenté comme la preuve d’une résiliation fournisseur.
9. Un client ne peut accéder aux abonnements d’un autre client.
10. Toute mutation et toute consultation administrative sensible produisent un événement d’audit.
