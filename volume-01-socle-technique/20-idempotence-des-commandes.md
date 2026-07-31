# Idempotence des commandes

## Objectif

Les paiements, transferts, encaissements, remboursements et opérations administratives peuvent être soumis plusieurs fois à cause d’une reprise réseau, d’un délai d’attente côté client, d’un webhook redélivré ou d’une nouvelle tentative automatique. Mansa doit garantir qu’une même commande logique ne produit pas plusieurs effets financiers.

La primitive `decideIdempotency` est exposée par `@mansa/domain` depuis `packages/domain/src/idempotency.ts`.

## Données minimales

Un enregistrement d’idempotence contient :

- `key` : clé stable fournie par le canal appelant ;
- `requestHash` : empreinte canonique de la requête protégée ;
- `responseReference` : référence du résultat lorsqu’il est disponible ;
- `expiresAt` : date d’expiration de la protection.

La clé et l’empreinte sont obligatoires et non vides. La date d’expiration doit être valide. Une référence de réponse vide est interdite.

## Décisions métier

La fonction retourne l’une des décisions suivantes :

- `ACCEPT` : aucune commande active équivalente n’existe, ou l’enregistrement précédent est expiré ;
- `REPLAY` : la même clé et la même requête ont déjà terminé, la réponse précédente doit être rejouée ;
- `CONFLICT` : la clé est déjà utilisée pour une autre requête, ou la première exécution n’a pas encore produit de réponse réutilisable.

Une clé inconnue est acceptée. Une clé expirée peut être réutilisée. Une clé active ne doit jamais autoriser deux effets financiers concurrents.

## Empreinte canonique

Le `requestHash` doit être calculé sur une représentation canonique et stable de la commande. Les champs non déterministes, tels qu’un horodatage technique généré par le serveur, ne doivent pas modifier l’empreinte.

L’empreinte doit couvrir au minimum les éléments qui changent l’effet métier :

- type d’opération ;
- identifiants du payeur et du bénéficiaire ;
- montant en unité mineure et devise ;
- canal, pays et partenaire ;
- références métier obligatoires ;
- paramètres de frais ou de partage lorsqu’ils sont fournis par le client.

L’algorithme de hachage et la version de canonisation doivent être documentés dans l’adaptateur d’entrée.

## Persistance et concurrence

La lecture de la clé et sa réservation doivent être atomiques. Deux requêtes concurrentes utilisant la même clé ne doivent pas obtenir simultanément `ACCEPT`.

L’implémentation persistante doit utiliser une contrainte unique sur le périmètre choisi, par exemple :

```text
(tenant_id, channel, idempotency_key)
```

Le premier traitement crée un enregistrement sans `responseReference`. Une fois l’opération terminée, le résultat durable est attaché au même enregistrement. Une nouvelle tentative pendant l’exécution reçoit un conflit temporaire ou une réponse explicite de traitement en cours, sans relancer l’effet financier.

## Périmètre des clés

La même chaîne peut être utilisée par deux tenants ou deux canaux uniquement si le périmètre de stockage les distingue. Le périmètre doit être fixé avant la mise en production et rester cohérent entre API publique, applications internes, webhooks et traitements asynchrones.

Une clé ne doit pas contenir de secret, de donnée KYC ni de donnée bancaire sensible.

## Durée de conservation

La durée de protection dépend du produit et du canal. Elle doit être configurable, mais suffisamment longue pour couvrir les reprises réseau, redélivrances de webhooks, délais des partenaires et rapprochements opérationnels.

L’expiration d’une clé ne supprime pas les écritures financières ni les journaux d’audit associés. Elle autorise seulement une nouvelle décision d’idempotence pour cette clé.

## Réponses et erreurs

- `REPLAY` doit retourner le même statut métier et la même référence durable que l’exécution initiale.
- `CONFLICT` pour une empreinte différente doit être distingué d’une erreur technique.
- Une exécution en cours ne doit pas être présentée comme un échec financier définitif.
- Une erreur avant tout effet durable peut permettre une nouvelle tentative contrôlée.
- Une erreur après un effet durable doit conserver la clé et rendre le résultat récupérable.

## Journalisation et sécurité

Les journaux doivent contenir la clé sous une forme compatible avec les règles de confidentialité, la décision obtenue, la référence de commande et le canal. Le corps intégral de la requête ne doit pas être journalisé lorsqu’il contient des données sensibles.

Les métriques minimales sont : nombre de décisions `ACCEPT`, `REPLAY` et `CONFLICT`, conflits par canal, âge des traitements en cours et taux de rejeu.

## Critères d’acceptation

- Une clé absente du stockage retourne `ACCEPT`.
- La même clé, la même empreinte et une réponse existante retournent `REPLAY` avec la référence d’origine.
- La même clé avec une empreinte différente retourne `CONFLICT`.
- La même clé encore en cours retourne `CONFLICT` et ne relance pas l’opération.
- Une clé expirée retourne `ACCEPT`.
- Une clé, une empreinte ou une date invalide est rejetée.
- Les tests automatisés couvrent les décisions, les données invalides et l’expiration.
- Les futurs adaptateurs de persistance devront couvrir les courses concurrentes et la contrainte unique.

## Cohérence avec le code

Le contrat actuellement disponible dans `@mansa/domain` couvre la décision pure et la validation de l’enregistrement. La persistance atomique, le calcul canonique de l’empreinte, les codes HTTP, la récupération de réponse et les métriques restent à implémenter dans les couches application et infrastructure.
