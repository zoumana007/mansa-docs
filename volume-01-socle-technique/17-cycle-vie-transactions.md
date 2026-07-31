# Cycle de vie des transactions

## Objectif

Le cycle de vie impose des transitions déterministes et auditables pour toutes les opérations financières Mansa. Une transaction ne doit jamais passer directement d’un état initial à un succès final sans avoir franchi les étapes applicables au type d’opération et aux intégrations partenaires.

## États canoniques

- `PENDING` : commande acceptée mais non encore autorisée ou traitée.
- `AUTHORIZED` : fonds, moyen de paiement ou partenaire autorisé.
- `PROCESSING` : traitement financier ou partenaire en cours.
- `SUCCEEDED` : opération confirmée avec succès.
- `FAILED` : échec définitif avant succès.
- `CANCELLED` : annulation définitive avant succès.
- `REVERSED` : opération initialement réussie puis compensée ou annulée financièrement.

## Transitions autorisées

| État source | États cibles autorisés |
| --- | --- |
| `PENDING` | `AUTHORIZED`, `PROCESSING`, `FAILED`, `CANCELLED` |
| `AUTHORIZED` | `PROCESSING`, `FAILED`, `CANCELLED` |
| `PROCESSING` | `SUCCEEDED`, `FAILED`, `CANCELLED` |
| `SUCCEEDED` | `REVERSED` |
| `FAILED` | aucune |
| `CANCELLED` | aucune |
| `REVERSED` | aucune |

Les états `FAILED`, `CANCELLED` et `REVERSED` sont terminaux. `SUCCEEDED` est final pour le traitement nominal, mais autorise une transition compensatoire vers `REVERSED`.

## Règles métier

1. Chaque transition doit être validée par le domaine avant toute écriture persistante.
2. Une transition invalide est rejetée sans modifier les soldes, le ledger ni les événements.
3. Chaque transition persistée produit un événement d’audit contenant la référence publique, l’état source, l’état cible, la date UTC, l’acteur ou service et l’identifiant de corrélation.
4. Un rejeu idempotent d’une commande déjà appliquée retourne l’état courant sans dupliquer les écritures comptables.
5. Un succès ne peut être déclaré qu’après confirmation du ledger interne et, lorsqu’un partenaire externe intervient, après confirmation conforme de ce partenaire.
6. Une inversion après succès utilise `REVERSED` et des écritures compensatoires ; elle ne supprime ni ne modifie rétroactivement les écritures originales.
7. Les délais, reprises et webhooks tardifs doivent être traités explicitement par les cas d’usage applicatifs sans contourner la machine d’états.

## Implémentation de référence

Le package `@mansa/domain` expose :

- `allowedTransactionTransitions` pour consulter les états cibles possibles ;
- `canTransitionTransaction` pour vérifier une transition sans exception ;
- `assertTransactionTransition` pour imposer la règle dans un cas d’usage ;
- `InvalidTransactionTransitionError` pour une erreur métier structurée.

Le domaine reste indépendant de NestJS, Prisma, HTTP et des fournisseurs externes.

## Critères d’acceptation

- Les transitions nominales `PENDING → AUTHORIZED → PROCESSING → SUCCEEDED` sont acceptées.
- `PENDING → PROCESSING` est accepté pour les rails sans étape d’autorisation distincte.
- `SUCCEEDED → REVERSED` est accepté.
- Un passage direct `PENDING → SUCCEEDED` est refusé.
- Aucun état terminal ne peut revenir vers un état actif.
- La liste des transitions exposée par le domaine est immuable.
- Les tests automatisés couvrent transitions valides, invalides, états terminaux et erreur détaillée.

## Suite d’implémentation

- Appliquer la validation dans les cas d’usage de paiement, transfert, cash-in, cash-out et remboursement.
- Persister l’historique des transitions séparément de l’état courant.
- Définir les politiques de délais et de reprise par rail de paiement.
- Mapper les statuts partenaires vers les états canoniques sans exposer leurs particularités au domaine.
- Ajouter des métriques sur le temps passé dans chaque état et les transactions bloquées en `PROCESSING`.
