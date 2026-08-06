# Catalogue des contrats API partagés

## Objectif

Le package `@mansa/contracts` constitue la source commune des types métier, commandes, réponses, routes et méthodes HTTP utilisées par les applications et services Mansa. Les consommateurs doivent importer un domaine précis plutôt que recopier localement les DTO.

## Points d’entrée

Le point d’entrée agrégé est :

```ts
import type { PaymentApiContract } from '@mansa/contracts/api-contracts';
```

Pour réduire les dépendances et améliorer la lisibilité, les applications peuvent utiliser un sous-chemin spécialisé :

```ts
import type { PaymentApiContract } from '@mansa/contracts/payment-api';
import type { NotificationApiContract } from '@mansa/contracts/notification-api';
import type { SupportApiContract } from '@mansa/contracts/support-api';
```

## Domaines exposés

| Domaine | Sous-chemin public | Responsabilité principale |
|---|---|---|
| Identité | `@mansa/contracts/identity-api` | inscription, authentification, sessions et vérifications |
| KYC | `@mansa/contracts/kyc-api` | dossiers, documents, soumission et revue |
| Grand livre | `@mansa/contracts/ledger-api` | journaux, écritures, soldes et annulations compensatoires |
| Portefeuilles | `@mansa/contracts/wallet-api` | comptes de paiement et consultations de solde |
| Paiements | `@mansa/contracts/payment-api` | paiements, demandes de paiement et statuts |
| Transferts | `@mansa/contracts/transfer-api` | devis, création, autorisation et annulation |
| Cartes | `@mansa/contracts/card-api` | émission, contrôles, limites et changements de statut |
| Commerçants | `@mansa/contracts/merchant-api` | commerces, établissements, membres et tableaux de bord |
| Terminaux | `@mansa/contracts/terminal-api` | enrôlement, activation, configuration et santé TPE |
| Services publics | `@mansa/contracts/public-services-api` | obligations, paiements, bourses et cartes étudiantes |
| Notifications | `@mansa/contracts/notification-api` | envoi multicanal, suivi, annulation et relance |
| Support | `@mansa/contracts/support-api` | tickets, messages, affectation et mise à jour |
| Bénéficiaires | `@mansa/contracts/beneficiary-api` | création, vérification et gestion des bénéficiaires |
| Annuaire | `@mansa/contracts/directory-api` | profils, recherche géographique et publication |
| Fidélité | `@mansa/contracts/loyalty-api` | programmes, points, récompenses et avantages |
| Administration | `@mansa/contracts/admin-api` | configuration, gouvernance et opérations administratives |
| Intégrations | `@mansa/contracts/integration-api` | partenaires, adaptateurs, capacités et état des connexions |
| Audit | `@mansa/contracts/audit-api` | recherche et consultation des événements d’audit |
| Webhooks | `@mansa/contracts/webhook-api` | abonnements, livraisons, signatures et relances |
| Remboursements | `@mansa/contracts/refund-api` | création, traitement et suivi des remboursements |
| Litiges | `@mansa/contracts/dispute-api` | ouverture, preuves, décisions et clôture |
| Réconciliation | `@mansa/contracts/reconciliation-api` | rapprochement interne et partenaire |
| Règlement | `@mansa/contracts/settlement-api` | lots, destinations, exécution et états de règlement |
| Intelligence | `@mansa/contracts/intelligence-api` | Jini, assistance et décisions contrôlées |
| Analytics | `@mansa/contracts/analytics-api` | indicateurs, séries, rapports et exports |

## Règles de conception

1. Toute route publique est versionnée sous `/v1` tant qu’aucune version ultérieure n’est introduite.
2. Toute commande financière ou action sensible possède une clé d’idempotence.
3. Les réponses paginées utilisent les contrats partagés de pagination.
4. Les dates sont transportées en chaînes ISO 8601 en UTC.
5. Les montants utilisent des unités mineures entières et un code devise explicite.
6. Les contrats ne contiennent aucun secret, jeton, numéro de carte complet ni document KYC brut.
7. Les identifiants de corrélation doivent traverser la passerelle, les services, les workers et les adaptateurs.
8. Une modification incompatible exige une nouvelle version de route ou une migration coordonnée.

## Cohérence avec le code

Le fichier `packages/contracts/src/api-contracts.ts` agrège les contrats HTTP. Le champ `exports` de `packages/contracts/package.json` doit exposer chaque sous-chemin documenté ci-dessus. Toute nouvelle API doit mettre à jour simultanément :

- son fichier de contrat ;
- l’agrégateur `api-contracts.ts` ;
- les exports du package ;
- ce catalogue ;
- les tests de type et les critères de recette associés.

## Validation minimale

Avant fusion, exécuter depuis la racine de `mansa-platform` :

```bash
pnpm --filter @mansa/contracts typecheck
pnpm --filter @mansa/contracts build
```

Une validation est refusée lorsqu’un sous-chemin pointe vers un fichier absent, lorsqu’un contrat utilise un type non exporté, ou lorsque la documentation annonce une route inexistante.
