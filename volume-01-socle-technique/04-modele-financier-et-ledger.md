# Volume 1 — Modèle financier et grand livre

## 1. Objectif

Le grand livre constitue la source de vérité de tous les mouvements financiers de Mansa. Aucun solde affiché, aucune commission et aucun rapprochement ne doit être calculé uniquement à partir d’un état mutable sans lien avec les écritures comptables correspondantes.

## 2. Principes obligatoires

- Comptabilité en partie double.
- Somme des débits égale à la somme des crédits pour chaque transaction validée.
- Montants stockés en unités mineures entières.
- Devise obligatoire sur chaque compte et écriture.
- Écritures validées immuables.
- Correction par contre-écriture, jamais par modification silencieuse.
- Idempotence obligatoire pour toute commande pouvant créer un mouvement.
- Traçabilité complète entre intention, transaction, écritures, partenaire et événement.

## 3. Objets principaux

### Compte de grand livre

Un compte représente un compartiment comptable identifié par :

- un identifiant opaque ;
- un propriétaire métier éventuel ;
- une devise ;
- un type de compte ;
- un statut ;
- une politique de découvert ;
- un environnement et un pays ;
- des dates de création et de clôture.

Types minimaux : actif client, disponible client, réserve, commerçant à payer, commission Mansa, frais partenaire, compte de règlement, suspense, chargeback, remboursement et compte technique de rapprochement.

### Transaction financière

Une transaction regroupe une opération métier unique. Elle contient au minimum :

- identifiant public ;
- type et version du flux ;
- statut ;
- clé d’idempotence ;
- montant demandé et devise ;
- acteur initiateur ;
- références métier et partenaire ;
- identifiant de corrélation ;
- horodatages ;
- motif d’échec ou d’annulation lorsque applicable.

### Écriture

Chaque écriture contient :

- transaction ;
- compte ;
- sens débit ou crédit ;
- montant entier strictement positif ;
- devise ;
- séquence ;
- libellé métier ;
- date comptable.

## 4. Soldes

Trois notions sont distinguées :

- **solde comptable** : résultat des écritures validées ;
- **solde disponible** : montant utilisable après réservations et restrictions ;
- **solde réservé** : fonds bloqués pour une opération non finalisée.

Les projections de solde peuvent être matérialisées pour la performance, mais doivent toujours être réconciliables avec les écritures du grand livre.

## 5. Cycle d’une opération

1. Réception de la commande et contrôle de la clé d’idempotence.
2. Validation du compte, de la devise, des limites et du statut réglementaire.
3. Création d’une intention de transaction.
4. Réservation éventuelle des fonds.
5. Appel du partenaire lorsque nécessaire.
6. Validation atomique des écritures équilibrées.
7. Libération ou conversion de la réservation.
8. Publication d’un événement via une outbox transactionnelle.
9. Mise à jour des projections et notifications.
10. Rapprochement avec les relevés partenaires.

## 6. Idempotence

Une clé d’idempotence est unique au minimum par acteur, route métier et environnement. Une nouvelle requête avec la même clé et le même contenu retourne le résultat initial. Une requête avec la même clé mais un contenu différent est rejetée et auditée.

Les webhooks partenaires utilisent une clé dérivée de l’identifiant externe, du partenaire et du type d’événement.

## 7. Réservations et expirations

Une réservation possède un montant, une devise, un compte source, un statut et une date d’expiration. Son expiration est traitée de façon idempotente. Une réservation finalisée ne peut pas être relâchée une seconde fois.

## 8. Frais et commissions

Les frais sont représentés par des écritures dédiées et non par une simple soustraction non tracée. Le calcul conserve :

- la règle tarifaire appliquée ;
- sa version ;
- la base de calcul ;
- le montant brut ;
- les taxes éventuelles ;
- le bénéficiaire ;
- les arrondis.

Une modification de tarification ne change jamais rétroactivement une transaction validée.

## 9. Conversion de devise

Une conversion crée au minimum deux ensembles d’écritures et conserve le taux, sa source, sa date, la marge, les arrondis et les comptes de compensation. Une opération multi-devise ne mélange jamais deux devises sur un même compte.

## 10. Annulation, remboursement et litige

- Une annulation avant règlement libère la réservation.
- Un remboursement après règlement crée une nouvelle transaction liée à l’originale.
- Un remboursement partiel conserve le cumul déjà remboursé.
- Un chargeback utilise des comptes dédiés et un état de litige.
- Une correction comptable crée une contre-écriture explicite avec motif et approbation.

## 11. Rapprochement

Chaque intégration financière doit fournir un mécanisme de rapprochement entre :

- transactions internes ;
- écritures du grand livre ;
- événements et webhooks ;
- fichiers ou API du partenaire ;
- mouvements du compte de règlement.

Les écarts sont classés, assignés et résolus sans altérer les écritures historiques.

## 12. Contrôles automatiques

- Rejet d’une transaction déséquilibrée.
- Rejet d’un montant nul ou négatif dans une ligne d’écriture.
- Rejet d’une devise incompatible avec le compte.
- Unicité de la clé d’idempotence dans son périmètre.
- Interdiction de modifier ou supprimer une écriture validée.
- Vérification périodique de l’équilibre global.
- Détection des soldes interdits et comptes techniques non apurés.

## 13. Critères d’acceptation

- Deux appels identiques avec la même clé ne créent qu’une transaction.
- Une transaction validée possède au moins deux écritures et est équilibrée.
- Un échec partenaire ne crée pas de débit définitif sans état compensable.
- Un remboursement partiel ne peut pas dépasser le montant remboursable.
- Les soldes projetés peuvent être reconstruits depuis les écritures.
- Toute correction conserve l’opération originale et le lien vers la contre-écriture.
- Les écarts de rapprochement sont visibles sans accès direct à la base.
