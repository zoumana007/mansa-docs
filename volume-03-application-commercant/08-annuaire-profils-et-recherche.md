# Annuaire commerçant, profils publics et recherche

## 1. Objectif

L’annuaire Mansa permet à un commerce de publier un profil public consultable dans l’application Annuaire, l’application Client et les surfaces web autorisées. Il sert à découvrir un commerce, consulter ses informations utiles, lancer un contact, ouvrir un itinéraire ou initier une action Mansa sans exposer de données privées.

## 2. Frontières du module

Le module Annuaire gère :

- les profils publics liés à un commerçant et, facultativement, à un point de vente ;
- les catégories, étiquettes, descriptions, horaires et coordonnées publiques ;
- la position géographique et la recherche par proximité ;
- les médias référencés par identifiants d’actifs ;
- les niveaux d’abonnement et la mise en avant ;
- la modération, la publication, la suspension et l’archivage.

Il ne gère pas directement les fichiers médias, la facturation des abonnements, les paiements ou le calcul d’itinéraire. Ces fonctions passent par leurs services respectifs.

## 3. Modèle de profil

Un profil contient au minimum :

- un identifiant interne ;
- le commerçant propriétaire ;
- un slug public unique ;
- un nom d’affichage ;
- une description courte ;
- au moins une catégorie ;
- un pays ;
- un statut de publication ;
- un niveau d’abonnement ;
- les dates de création et de modification.

Les champs facultatifs comprennent la description longue, la ville, la zone administrative, l’adresse, la géolocalisation, les contacts, les horaires, le logo, la couverture et la galerie.

## 4. Cycle de vie

Les statuts autorisés sont :

- `DRAFT` : profil non visible, modifiable par son propriétaire ;
- `PENDING_REVIEW` : profil soumis à la modération ;
- `PUBLISHED` : profil visible selon les règles de ciblage ;
- `SUSPENDED` : profil temporairement masqué avec justification ;
- `ARCHIVED` : profil définitivement retiré du catalogue actif.

Transitions minimales :

- `DRAFT` vers `PENDING_REVIEW` ou `ARCHIVED` ;
- `PENDING_REVIEW` vers `DRAFT`, `PUBLISHED`, `SUSPENDED` ou `ARCHIVED` ;
- `PUBLISHED` vers `SUSPENDED` ou `ARCHIVED` ;
- `SUSPENDED` vers `PENDING_REVIEW`, `PUBLISHED` ou `ARCHIVED` ;
- aucune sortie directe depuis `ARCHIVED`.

Toute suspension, remise en publication ou archive administrative doit produire un événement d’audit avec acteur, motif, date et identifiant de corrélation.

## 5. Contacts et confidentialité

Les canaux autorisés sont `PHONE`, `EMAIL`, `WHATSAPP`, `WEBSITE` et `IN_APP`.

Un contact doit préciser s’il est principal. Les API privées peuvent transporter une valeur masquée. Les API publiques ne renvoient une valeur complète que lorsque le commerçant l’a explicitement déclarée publique. Aucun numéro personnel issu du KYC, aucune adresse privée et aucun secret d’intégration ne doivent être copiés automatiquement dans le profil.

## 6. Géolocalisation

Les coordonnées doivent respecter : latitude entre -90 et 90, longitude entre -180 et 180. La recherche par rayon exige latitude, longitude et rayon positif. Le backend applique une limite maximale de rayon configurable par pays et par canal afin d’éviter les recherches coûteuses ou abusives.

Les résultats peuvent fournir une distance en mètres et un score de pertinence. Ces deux valeurs sont calculées par le service de recherche et ne sont jamais acceptées depuis le client comme données fiables.

## 7. Horaires

Chaque période d’ouverture contient un jour de semaine et deux heures locales. Le pays et le point de vente déterminent le fuseau applicable. Les périodes qui traversent minuit doivent être normalisées en deux périodes ou prises en charge explicitement par le moteur d’horaires. La recherche « ouvert à » doit être évaluée côté serveur.

## 8. Abonnements et mise en avant

Les niveaux initiaux sont `FREE`, `STANDARD`, `PREMIUM` et `ENTERPRISE`. Ils ne donnent aucun droit implicite dans le code client : les capacités effectives sont résolues depuis la configuration commerciale active.

La mise en avant possède une date de fin. Un profil ne doit plus être classé comme mis en avant après cette date, même si un indicateur obsolète reste présent. Les changements commerciaux doivent être audités et rapprochés du service de facturation.

## 9. Recherche

Les filtres pris en charge sont : texte, catégories, étiquettes, pays, zone administrative, ville, proximité, horaire et mise en avant. La pagination est obligatoire. Les profils non publiés sont exclus des recherches publiques.

Le classement combine au minimum : pertinence textuelle, proximité lorsque disponible, qualité du profil, disponibilité et mise en avant autorisée. Le classement sponsorisé doit rester identifiable et ne doit jamais contourner une suspension ou une règle réglementaire.

## 10. Contrats techniques correspondants

Le dépôt `mansa-platform` expose :

- `packages/contracts/src/directory.ts` pour le modèle, les commandes, les transitions et la validation géographique ;
- `packages/contracts/src/directory-api.ts` pour les routes et contrats HTTP ;
- les sous-chemins `@mansa/contracts/directory` et `@mansa/contracts/directory-api` pour les consommateurs du monorepo.

Les applications ne doivent pas redéfinir localement ces statuts ou structures.

## 11. Sécurité et modération

- Autorisation par propriétaire, membre du commerce ou rôle administratif explicite.
- Idempotence obligatoire pour la création et les changements de statut.
- Contrôle antivirus et validation de type pour tous les médias référencés.
- Limitation de débit sur la recherche publique et les soumissions.
- Journalisation sans valeur de contact complète.
- Détection des slugs trompeurs, contenus interdits et usurpations de marque.
- Suspension immédiate possible depuis l’administration, sans suppression des preuves.

## 12. Critères d’acceptation

1. Un commerçant autorisé crée un brouillon avec une clé d’idempotence.
2. Un profil sans catégorie, pays ou description courte est refusé.
3. Une coordonnée hors limites est refusée.
4. Un brouillon n’apparaît jamais dans la recherche publique.
5. Une transition interdite retourne une erreur métier stable.
6. Un profil suspendu disparaît des recherches et accès publics.
7. Une recherche géographique renvoie uniquement des profils dans le rayon autorisé.
8. Les valeurs privées des contacts sont masquées dans les journaux et réponses non autorisées.
9. Une mise en avant expirée n’influence plus le classement.
10. Les changements sensibles sont corrélés à un événement d’audit.
