# Attribution d’un identifiant Mansa

## Statut

- Domaine : application Client / Mansa Connect
- Implémentation de référence : `mansa-platform/packages/domain/src/mansa-handle-directory.ts`
- Dépendance : contrat de validation décrit dans `02-contrat-identifiant-mansa.md`

## Objectif

Le service d’attribution associe un identifiant Mansa validé à un propriétaire interne sans dépendre d’un framework, d’une base de données ou d’un fournisseur d’identité.

## Propriétaires pris en charge

Le socle distingue quatre catégories :

- `USER` : particulier ;
- `MERCHANT` : commerce ;
- `AGENT` : agent terrain ou agent public ;
- `ORGANIZATION` : entreprise, banque, université ou administration.

Chaque propriétaire est identifié par la combinaison immuable `type + id`.

## Invariants du socle

1. Un identifiant Mansa ne peut être attribué qu’une seule fois.
2. Un propriétaire ne peut posséder qu’un seul identifiant actif.
3. L’identifiant interne du propriétaire est nettoyé de ses espaces périphériques et ne peut pas être vide.
4. La date d’attribution provient d’une horloge injectable afin de rendre le comportement testable.
5. L’objet retourné est figé et la date enregistrée est copiée avant exposition.
6. La sauvegarde est déléguée à un port `MansaHandleDirectory`.

## Port de persistance

L’adaptateur de stockage doit implémenter :

- `findByHandle(handle)` pour rechercher une attribution par valeur normalisée ;
- `findByOwner(owner)` pour rechercher l’identifiant actif du propriétaire ;
- `save(assignment)` pour enregistrer l’attribution.

En production, les lectures préalables ne remplacent pas les contraintes uniques de la base. L’adaptateur doit imposer au minimum :

- une unicité sur la valeur normalisée de l’identifiant ;
- une unicité sur la paire `ownerType + ownerId` pour les attributions actives ;
- une transaction atomique entre vérification, insertion et événement d’audit.

## Erreurs métier

`MansaHandleAlreadyAssignedError` couvre deux conflits :

- l’identifiant demandé appartient déjà à un autre propriétaire ;
- le propriétaire possède déjà un identifiant actif.

L’API traduira ces conflits en réponse fonctionnelle stable, sans exposer de détails internes ni permettre l’énumération abusive des comptes.

## Concurrence et idempotence

Deux requêtes simultanées peuvent réussir les vérifications en mémoire. La base doit donc arbitrer définitivement avec ses contraintes uniques. Le service applicatif supérieur devra aussi accepter une clé d’idempotence pour rejouer sans créer une seconde attribution.

## Audit attendu

Toute attribution réussie doit produire un événement contenant au minimum :

- l’identifiant normalisé ;
- le type et l’identifiant interne du propriétaire ;
- l’horodatage ;
- l’acteur ou le canal à l’origine de la demande ;
- l’identifiant de corrélation et la clé d’idempotence.

Aucune donnée secrète, aucun jeton et aucun document KYC ne doivent être inclus dans cet événement.

## Critères de recette

1. `@zoumana` peut être associé à `USER:user-007` lorsqu’aucune attribution n’existe.
2. Les espaces de l’identifiant interne sont supprimés avant persistance.
3. Une seconde attribution de `@zoumana` est refusée.
4. Un second identifiant pour `USER:user-007` est refusé.
5. Un identifiant interne vide est refusé.
6. Les tests du package domaine utilisent un dépôt en mémoire et une horloge fixe.
