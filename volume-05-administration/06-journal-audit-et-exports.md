# Journal d’audit et exports contrôlés

## Objectif

Le journal d’audit fournit une preuve exploitable des actions sensibles réalisées dans Mansa. Il complète les journaux techniques : il décrit qui a agi, sur quelle ressource, avec quel résultat et dans quel contexte métier.

## Événement d’audit minimal

Chaque événement contient :

- un identifiant unique ;
- la date et l’heure UTC de l’action ;
- l’acteur et son type (`USER`, `ADMIN`, `SERVICE` ou `PARTNER`) ;
- le rôle et l’organisation lorsqu’ils existent ;
- l’action normalisée ;
- le type et l’identifiant de la ressource ;
- le résultat (`SUCCESS`, `FAILURE` ou `DENIED`) ;
- la raison d’un refus ou d’un échec lorsqu’elle est disponible ;
- l’identifiant de corrélation ;
- le pays, l’appareil, l’adresse IP et l’agent utilisateur lorsque leur collecte est autorisée ;
- des métadonnées limitées à des valeurs non secrètes.

Aucun mot de passe, OTP, code PIN, secret d’API, numéro de carte complet, document KYC brut ou donnée biométrique ne doit être inscrit dans les événements.

## Actions obligatoirement auditées

- connexion, échec de connexion, révocation de session et changement de facteur ;
- création, suspension, réactivation et clôture de compte ;
- décision KYC et modification d’un niveau de vérification ;
- création, publication, annulation et compensation d’un journal financier ;
- modification de frais, commissions, limites, routes de paiement ou paramètres pays ;
- activation ou blocage d’une carte, d’un terminal, d’un commerçant ou d’un partenaire ;
- décision d’approbation à double contrôle ;
- consultation ou export de données sensibles ;
- action d’un agent public sur une obligation, une bourse, une amende ou une carte étudiante ;
- changement de rôle, de permission, de politique ou de drapeau de fonctionnalité.

## Consultation

L’API de consultation est réservée aux rôles autorisés. Les filtres minimaux portent sur l’acteur, l’organisation, l’action, la ressource, le résultat, la corrélation, le pays et la période.

Les résultats sont paginés. Le tri par défaut est antéchronologique, avec un ordre stable permettant de parcourir un grand volume sans doublon.

## Exports

Les exports sont asynchrones et limités aux formats `CSV` et `JSONL`.

Toute demande d’export exige :

1. un motif explicite ;
2. une clé d’idempotence ;
3. une autorisation adaptée au périmètre ;
4. une double validation lorsque le périmètre ou la sensibilité l’exige ;
5. un événement d’audit pour la demande et pour le téléchargement.

Un export suit les statuts `PENDING`, `PROCESSING`, `READY`, `FAILED` ou `EXPIRED`. La réponse ne contient pas directement les données : elle expose une référence d’objet temporaire générée par le service de stockage sécurisé. L’URL de téléchargement éventuelle est courte, signée et non journalisée.

## Intégrité et conservation

- Les événements publiés sont immuables.
- Les corrections produisent de nouveaux événements liés aux précédents.
- Les écritures sont horodatées côté serveur.
- Les permissions d’écriture directe dans le stockage d’audit sont interdites aux applications clientes.
- La durée de conservation est configurée par pays et catégorie, après validation juridique.
- Une empreinte ou un chaînage cryptographique peut être ajouté pour détecter les altérations.
- Les sauvegardes, restaurations et purges légales sont elles-mêmes auditées.

## Contrat technique correspondant

Le dépôt `mansa-platform` définit :

- le modèle commun dans `packages/contracts/src/audit.ts` ;
- les routes et DTO dans `packages/contracts/src/audit-api.ts` ;
- l’agrégation des contrats dans `packages/contracts/src/api-contracts.ts`.

Routes initiales :

- `GET /v1/audit/events` ;
- `GET /v1/audit/events/:eventId` ;
- `POST /v1/audit/exports` ;
- `GET /v1/audit/exports/:exportId`.

## Critères d’acceptation

- Une action sensible génère exactement un événement métier corrélable, même en cas de refus.
- Un utilisateur sans permission reçoit un refus sans fuite d’information et le refus est audité.
- Les filtres de période et d’organisation empêchent tout dépassement de périmètre.
- Deux demandes d’export avec la même clé d’idempotence ne créent pas deux exports.
- Un export expiré ne peut plus être téléchargé.
- Les métadonnées sont rejetées ou nettoyées lorsqu’elles contiennent un champ interdit.
- Les journaux applicatifs ne contiennent ni référence de stockage signée ni donnée sensible brute.
