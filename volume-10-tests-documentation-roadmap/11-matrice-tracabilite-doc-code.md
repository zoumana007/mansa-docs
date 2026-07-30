# Matrice de traçabilité documentation ↔ code

## Objectif

Cette matrice définit la méthode obligatoire pour vérifier que chaque exigence fonctionnelle documentée possède un emplacement de code, un contrat public et une preuve de validation. Elle évite qu’une fonctionnalité soit décrite sans être implémentable ou qu’un module soit ajouté sans spécification.

## Identifiant d’exigence

Chaque exigence détaillée utilise un identifiant stable au format :

```text
MANSA-<DOMAINE>-<NUMERO>
```

Exemples :

- `MANSA-AUTH-001` : connexion sécurisée ;
- `MANSA-LEDGER-001` : écritures équilibrées ;
- `MANSA-PAY-001` : idempotence d’un paiement ;
- `MANSA-ADMIN-001` : audit des actions sensibles.

Un identifiant n’est jamais réutilisé après suppression. Une exigence remplacée est marquée `superseded` et pointe vers son successeur.

## Colonnes obligatoires

| Champ | Description |
|---|---|
| Identifiant | Référence stable de l’exigence |
| Volume | Document fonctionnel source |
| Domaine | Module métier responsable |
| Contrat | DTO, schéma, événement ou route publique |
| Implémentation | Application ou paquet responsable |
| Données | Tables, agrégats ou projections concernées |
| Tests | Tests unitaires, intégration, contrat ou E2E |
| Statut | planned, scaffolded, implemented, verified, blocked |
| Preuve | Commande CI, rapport ou lien vers scénario de recette |
| Notes | Dépendances, limites ou configuration manuelle |

## Règles de cohérence

1. Une exigence `implemented` doit référencer au moins un fichier de code et un test.
2. Une exigence `verified` doit posséder une preuve reproductible.
3. Une route publique doit référencer un contrat versionné dans `packages/contracts`.
4. Une règle financière doit référencer le domaine et les tests du grand livre.
5. Une action d’administration sensible doit référencer une permission et un événement d’audit.
6. Une intégration externe doit référencer un port métier, un adaptateur et un scénario d’échec.
7. Les fonctionnalités non disponibles en production restent derrière un drapeau explicite.

## Lot initial de traçabilité

| Identifiant | Volume | Domaine | Contrat cible | Implémentation cible | Tests requis | Statut |
|---|---|---|---|---|---|---|
| MANSA-AUTH-001 | Volume 1/2 | identité | contrats auth v1 | `apps/api-gateway`, `packages/security` | unité + intégration + E2E | scaffolded |
| MANSA-KYC-001 | Volume 2 | conformité | contrats KYC v1 | module KYC + adaptateur fournisseur | contrat + intégration | planned |
| MANSA-LEDGER-001 | Volume 1/8 | grand livre | commandes et événements ledger | `packages/domain`, API, Prisma | propriétés + intégration DB | scaffolded |
| MANSA-PAY-001 | Volume 2/3/4 | paiements | paiement v1 | API + workers + adaptateurs | idempotence + panne/reprise | planned |
| MANSA-MERCHANT-001 | Volume 3 | commerçants | commerçant v1 | API + mobile merchant | unité + E2E | planned |
| MANSA-TPE-001 | Volume 4 | TPE | terminal et transaction v1 | Android + API | matériel simulé + E2E | planned |
| MANSA-ADMIN-001 | Volume 5 | administration | permissions et audit v1 | admin web + API | permission + audit | planned |
| MANSA-STATE-001 | Volume 6 | services publics | amende v1 | API + interfaces agents | anti-double paiement + audit | planned |
| MANSA-AI-001 | Volume 7 | Jini | requête assistant v1 | AI services + API | sécurité + évaluation | planned |
| MANSA-REPORT-001 | Volume 8 | reporting | export v1 | workers + admin web | intégrité + autorisation | planned |
| MANSA-OPS-001 | Volume 9 | exploitation | événements observabilité | infra + observability | reprise + alerte | planned |

## Mise à jour

La matrice détaillée peut être fractionnée par domaine dans ce dépôt. Le dépôt `mansa-platform` conserve un manifeste technique miroir ne contenant que les chemins réels et les commandes de validation. Toute livraison fonctionnelle met à jour les deux côtés dans le même lot ou indique clairement pourquoi l’un des deux changements est différé.

## Critères d’acceptation

- Aucun module déclaré terminé ne possède d’exigence sans test associé.
- Les chemins de code référencés existent sur la branche principale.
- Les contrats publics sont versionnés et leur compatibilité est vérifiée.
- Le statut `verified` ne peut être attribué manuellement sans preuve de validation.
- La CI signale les références de fichiers devenues invalides dès que l’automatisation correspondante est disponible.
