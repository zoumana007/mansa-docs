# Annuaire commerçant et mini-site public

## Objectif

Le module Annuaire permet à chaque commerçant autorisé de publier une fiche professionnelle et un mini-site public depuis les mêmes données administrées dans Mansa. Il doit faciliter la découverte locale, la prise de contact et l’accès aux services du commerçant sans exposer de données privées.

## Périmètre

Le premier socle couvre :

- la création d’une fiche liée à un commerçant et, si nécessaire, à un point de vente ;
- un nom public, un identifiant d’URL unique, une description courte et une description détaillée ;
- les catégories, mots-clés, zone administrative, ville, adresse et coordonnées géographiques ;
- les contacts publics, horaires, logo, couverture et galerie ;
- les offres `FREE`, `STANDARD`, `PREMIUM` et `ENTERPRISE` ;
- la mise en avant temporaire ;
- la recherche textuelle, géographique et par catégorie ;
- la publication d’un mini-site accessible par un slug stable.

## Cycle de vie

Les statuts initiaux sont :

- `DRAFT` : fiche modifiable et invisible publiquement ;
- `PENDING_REVIEW` : fiche soumise aux contrôles automatiques ou manuels ;
- `PUBLISHED` : fiche consultable dans l’annuaire et sur le mini-site ;
- `SUSPENDED` : fiche temporairement masquée avec motif audité ;
- `ARCHIVED` : fiche retirée définitivement du parcours métier normal.

Une fiche archivée n’est pas remise en ligne directement. Une nouvelle révision ou une nouvelle fiche doit être créée selon la politique administrative.

## Données publiques et privées

Seules les valeurs explicitement marquées comme publiques sont affichées. Les numéros internes, adresses de connexion, identifiants personnels, données KYC, secrets, informations bancaires et données complètes de paiement restent interdits dans le profil public.

Les journaux et interfaces d’administration utilisent des valeurs masquées lorsque la valeur complète n’est pas nécessaire.

## Slug et adresse publique

Le slug est unique dans le périmètre de l’annuaire. Il est normalisé côté serveur et ne contient ni secret ni donnée personnelle sensible. Les redirections après changement de slug doivent être contrôlées afin d’éviter l’usurpation ou la réutilisation immédiate d’une ancienne adresse.

## Recherche

La recherche peut combiner :

- texte libre ;
- catégories et mots-clés ;
- pays, région administrative et ville ;
- distance autour d’un point géographique ;
- ouverture à une date et heure données ;
- filtre des profils mis en avant.

Le classement doit distinguer la pertinence organique de la mise en avant commerciale. Une promotion ne doit pas être présentée comme un résultat neutre.

## Géolocalisation

Les coordonnées sont validées côté serveur : latitude entre -90 et 90, longitude entre -180 et 180. Une position approximative peut être utilisée lorsque le commerçant ne souhaite pas publier l’emplacement exact.

Les calculs de distance sont réalisés côté backend ou par un service géospatial contrôlé. Les applications ne doivent pas décider seules qu’un commerce appartient à un rayon donné.

## Horaires

Chaque période d’ouverture indique le jour, l’heure de début et l’heure de fin. Le backend doit gérer les périodes multiples dans une journée, les fermetures exceptionnelles, les jours fériés et le fuseau horaire du point de vente.

## Abonnements et mise en avant

L’offre souscrite détermine les capacités configurables : nombre de photos, statistiques, personnalisation, catégories, campagnes et durée de mise en avant.

La date de fin de mise en avant est obligatoire pour toute promotion temporaire. L’expiration retire automatiquement le marquage sans supprimer la fiche.

## Modération

La publication peut exiger :

- un commerçant actif et vérifié ;
- des contenus conformes aux politiques Mansa ;
- l’absence de catégorie interdite ;
- la validation des médias ;
- la cohérence de l’adresse et des contacts ;
- une autorisation renforcée pour les profils sensibles ou officiels.

Toute suspension exige un motif, un acteur et une trace d’audit.

## API

Les routes initiales sont :

- `POST /v1/directory/profiles` ;
- `GET /v1/directory/profiles` ;
- `GET /v1/directory/profiles/:profileId` ;
- `PATCH /v1/directory/profiles/:profileId` ;
- `POST /v1/directory/profiles/:profileId/status` ;
- `GET /v1/directory/search` ;
- `GET /v1/directory/public/:slug`.

Les listes et résultats de recherche sont paginés. Les mutations sensibles utilisent une clé d’idempotence lorsque leur répétition pourrait créer un doublon ou une transition incohérente.

## Contrat technique

Le domaine partagé est défini dans `mansa-platform/packages/contracts/src/directory.ts`.

Le contrat d’API est défini dans `mansa-platform/packages/contracts/src/directory-api.ts` et agrégé dans `packages/contracts/src/api-contracts.ts`.

Les applications concernées sont `apps/mobile-directory`, `apps/mobile-merchant`, `apps/public-web` et `apps/business-web`. Le backend autoritatif reste `apps/api-gateway`.

## Sécurité et audit

- La modification d’une fiche exige un droit sur le commerçant et le point de vente concernés.
- Les changements de statut, de slug, de coordonnées publiques et de mise en avant sont audités.
- Les médias passent par un stockage sécurisé, une validation de type et une analyse adaptée.
- Les contenus publics sont encodés et nettoyés contre les injections.
- Les résultats ne doivent pas révéler l’existence d’un profil privé ou suspendu à un utilisateur non autorisé.
- Les métriques publiques sont agrégées pour éviter l’identification abusive d’un visiteur.

## Critères d’acceptation

1. Deux créations avec la même clé d’idempotence ne produisent pas deux fiches.
2. Deux profils actifs ne peuvent pas utiliser le même slug.
3. Un brouillon n’apparaît jamais dans la recherche publique.
4. Une fiche suspendue disparaît du mini-site et de la recherche publique.
5. Les coordonnées géographiques invalides sont rejetées.
6. La recherche par rayon ne renvoie que des profils publiés dans le périmètre calculé.
7. L’expiration d’une mise en avant conserve la fiche mais retire son statut promotionnel.
8. Une donnée privée non marquée publique n’est jamais renvoyée par la route publique.
9. Toute transition de statut sensible produit un événement d’audit.
10. Les routes, méthodes et types documentés correspondent aux contrats partagés du dépôt plateforme.
