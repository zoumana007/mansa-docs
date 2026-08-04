# Grand livre, soldes et comptabilité transactionnelle

## 1. Objectif

Le grand livre constitue la source de vérité financière de Mansa. Les soldes affichés dans les applications sont des projections calculées à partir d'écritures comptables persistées, et non des valeurs modifiées directement par les modules métier.

Le contrat technique correspondant se trouve dans `@mansa/contracts`, fichiers `ledger.ts` et `ledger-api.ts`.

## 2. Principes obligatoires

- Toute opération financière est enregistrée en partie double.
- Une transaction comporte au minimum deux écritures.
- Le total des débits est strictement égal au total des crédits.
- Toutes les écritures d'une transaction utilisent la même devise.
- Les montants sont exprimés en unités mineures entières.
- Une transaction validée n'est jamais modifiée ni supprimée.
- Une correction utilise une transaction d'annulation liée à l'originale.
- La publication est atomique : toutes les écritures sont enregistrées ou aucune ne l'est.
- Une clé d'idempotence empêche une double comptabilisation.
- La référence métier, la corrélation et le pays sont conservés avec la transaction.

## 3. Comptes

Les types normalisés sont :

- `ASSET` : actifs détenus ou contrôlés ;
- `LIABILITY` : dettes de la plateforme envers clients, commerçants ou partenaires ;
- `EQUITY` : capitaux propres et comptes de contrôle associés ;
- `REVENUE` : commissions et revenus acquis ;
- `EXPENSE` : frais et charges comptabilisées.

Un compte possède un code stable, une devise unique, un pays et un propriétaire éventuel. Les propriétaires admis sont la plateforme, un utilisateur, un commerçant, un partenaire ou un organisme public.

Les comptes système sont créés par configuration contrôlée. Ils ne peuvent pas être renommés, supprimés ou réaffectés depuis une interface métier ordinaire.

## 4. Écritures

Chaque écriture contient :

- le compte comptable ;
- le sens `DEBIT` ou `CREDIT` ;
- le montant et la devise ;
- la transaction d'origine ;
- l'ordre de l'écriture dans la transaction ;
- la date de comptabilisation ;
- une description facultative ne contenant aucune donnée sensible.

Le moteur refuse les écritures nulles ou négatives, les mélanges de devises, les comptes inexistants ou inactifs et les transactions non équilibrées.

## 5. Cycle de vie

Une transaction comptable utilise les statuts suivants :

1. `PENDING` : préparation ou réservation en cours ;
2. `POSTED` : écritures définitivement publiées ;
3. `REVERSED` : transaction neutralisée par une transaction inverse ;
4. `REJECTED` : transaction refusée avant publication.

Le passage à `POSTED` est irréversible. Le statut `REVERSED` ne signifie pas que les écritures initiales ont été supprimées : elles restent visibles avec le lien vers la transaction d'annulation.

## 6. Soldes

Le service expose au minimum :

- le solde disponible ;
- le montant en attente ;
- l'instant de calcul `asOf`.

Une projection de solde peut être utilisée pour les lectures rapides, à condition qu'elle soit reconstructible depuis le journal. Toute divergence détectée entre projection et écritures bloque les traitements concernés et déclenche une alerte de rapprochement.

## 7. API interne

Le grand livre n'est pas directement exposé aux applications publiques. Les modules Paiement, Transfert, Carte, TPE, Remboursement, Règlement et Services publics l'appellent via une API interne authentifiée.

| Action | Méthode | Route |
|---|---|---|
| Publier une transaction | `POST` | `/v1/internal/ledger/transactions` |
| Lire une transaction | `GET` | `/v1/internal/ledger/transactions/:transactionId` |
| Annuler une transaction | `POST` | `/v1/internal/ledger/transactions/:transactionId/reverse` |
| Lire un compte | `GET` | `/v1/internal/ledger/accounts/:accountId` |
| Lire un solde | `GET` | `/v1/internal/ledger/accounts/:accountId/balance` |
| Lister l'historique | `GET` | `/v1/internal/ledger/accounts/:accountId/entries` |

Les écritures sont paginées par curseur. Les appels d'écriture exigent une clé d'idempotence et un identifiant de corrélation.

## 8. Exemples de schémas

### Transfert interne de 10 000 XOF

- débit du passif portefeuille expéditeur : 10 000 XOF ;
- crédit du passif portefeuille destinataire : 10 000 XOF.

### Paiement commerçant avec commission

Pour un paiement de 10 000 XOF et une commission de 200 XOF :

- débit du passif portefeuille client : 10 000 XOF ;
- crédit du passif commerçant : 9 800 XOF ;
- crédit du compte de revenu commission : 200 XOF.

L'exemple illustre la logique comptable. Le choix précis des comptes, taxes, frais partenaires et règles de règlement dépend du plan comptable validé.

## 9. Sécurité et contrôle

- Seuls des services autorisés peuvent publier des écritures.
- Aucune interface administrateur ne permet d'éditer une écriture publiée.
- Les annulations sensibles nécessitent une permission dédiée et, selon le seuil, une double validation.
- La création et la modification du plan comptable sont auditées.
- Les requêtes de lecture sont filtrées par pays, entité et périmètre d'autorisation.
- Les métadonnées ne contiennent ni secret, ni pièce KYC, ni donnée de carte complète.
- Les sauvegardes et restaurations doivent préserver l'ordre, les identifiants et l'intégrité référentielle.

## 10. Événements attendus

- `ledger.transaction.posted` ;
- `ledger.transaction.reversed` ;
- `ledger.transaction.rejected` ;
- `ledger.balance.updated` ;
- `ledger.integrity.failed`.

Chaque événement comporte la version du schéma, l'identifiant de transaction, la référence métier, le pays, la devise, la corrélation et l'horodatage.

## 11. Critères d'acceptation

- Une transaction déséquilibrée est rejetée avant toute écriture persistée.
- Une transaction mélangeant plusieurs devises est rejetée.
- Deux appels avec la même clé d'idempotence retournent la même transaction.
- Deux appels avec une même clé mais un contenu différent provoquent une erreur de conflit.
- Une annulation crée des écritures inverses et conserve les écritures originales.
- Une annulation répétée ne crée pas plusieurs transactions inverses.
- Le total reconstruit depuis les écritures correspond au solde projeté.
- Les transactions publiées restent lisibles après restauration d'une sauvegarde.
- Les modules métier n'écrivent jamais directement dans les tables du grand livre.
- Le contrat TypeScript, l'API NestJS, le schéma de données et la documentation OpenAPI utilisent les mêmes statuts et routes.

## 12. Éléments à configurer avant production

Le plan comptable définitif, les comptes de cantonnement, les règles de reconnaissance des commissions, les taxes, les comptes partenaires, les horaires de clôture, les règles de règlement, les seuils d'approbation et les exports réglementaires doivent être validés avec la banque partenaire, les experts comptables et les autorités compétentes de chaque pays.
