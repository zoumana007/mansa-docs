# Déclarations réglementaires

## Objectif

Ce module formalise le cycle de préparation, revue, approbation et soumission d’une déclaration réglementaire issue d’un dossier de conformité. Il couvre les rapports d’activité suspecte, les déclarations liées à un seuil et les rapports périodiques.

Le contrat technique de référence se trouve dans `mansa-platform/packages/security/src/regulatory-report.ts`.

## Principes

1. Une déclaration est toujours créée à l’état `DRAFT`.
2. La préparation et l’approbation sont séparées selon le principe des quatre yeux.
3. Une soumission n’est possible qu’après approbation.
4. Toute soumission conserve une référence externe ou un accusé de réception.
5. Les états finaux ne peuvent plus être modifiés.
6. Le domaine ne transmet aucune donnée directement à une autorité : l’envoi réel appartient à un adaptateur sécurisé et juridiquement validé.
7. Chaque transition retourne une action d’audit stable.

## Types de rapport

- `SUSPICIOUS_ACTIVITY` : déclaration liée à une activité ou opération suspecte ;
- `THRESHOLD` : déclaration imposée par un seuil réglementaire ;
- `PERIODIC` : rapport périodique demandé par une autorité ou un partenaire agréé.

Ces catégories techniques doivent être adaptées aux obligations applicables dans chaque pays avant mise en production.

## États

- `DRAFT` : rapport en préparation ;
- `READY_FOR_REVIEW` : contenu prêt pour contrôle ;
- `APPROVED` : rapport approuvé par un acteur différent du préparateur ;
- `SUBMITTED` : envoi confirmé avec référence externe ;
- `REJECTED` : rapport refusé lors de la revue ;
- `CANCELLED` : rapport annulé avant approbation.

Les états `SUBMITTED`, `REJECTED` et `CANCELLED` sont finaux. Un rapport approuvé ne peut plus être annulé : toute correction doit suivre une procédure d’amendement séparée.

## Données minimales

Chaque rapport contient :

- un identifiant unique ;
- l’identifiant du dossier de conformité source ;
- le pays concerné ;
- le type de rapport ;
- le code de l’autorité destinataire ;
- un résumé non vide ;
- l’auteur et la date de préparation ;
- le réviseur et la date de revue après décision ;
- l’auteur, la date et la référence externe après soumission.

Le résumé ne doit contenir que les informations nécessaires au workflow. Les pièces, données d’identité et détails sensibles restent dans un stockage protégé avec contrôle d’accès renforcé.

## Transitions autorisées

1. `DRAFT` → `READY_FOR_REVIEW` avec `REQUEST_REVIEW` ;
2. `READY_FOR_REVIEW` → `APPROVED` avec `APPROVE` ;
3. `READY_FOR_REVIEW` → `REJECTED` avec `REJECT` ;
4. `APPROVED` → `SUBMITTED` avec `MARK_SUBMITTED` ;
5. `DRAFT` ou `READY_FOR_REVIEW` → `CANCELLED` avec `CANCEL`.

Toute autre transition est refusée par le domaine.

## Principe des quatre yeux

Le préparateur ne peut pas approuver ou rejeter son propre rapport. Le service applicatif doit en plus vérifier :

- la permission réglementaire adaptée ;
- le périmètre pays et organisation ;
- le niveau d’habilitation ;
- l’absence de conflit d’intérêts ;
- l’authentification renforcée en Production ;
- la présence des justificatifs obligatoires.

## Soumission externe

L’adaptateur de soumission doit :

1. charger uniquement une version approuvée ;
2. générer un paquet conforme au format attendu ;
3. chiffrer le transport et authentifier le destinataire ;
4. utiliser une clé d’idempotence ;
5. conserver l’accusé de réception sans stocker de secret dans le dépôt ;
6. marquer le rapport `SUBMITTED` uniquement après confirmation vérifiable ;
7. envoyer les erreurs vers une file de reprise contrôlée ;
8. journaliser les métadonnées sans exposer le contenu sensible.

## Audit

Les actions suivantes doivent être enregistrées :

- `REGULATORY_REPORT_CREATED` ;
- `REGULATORY_REPORT_REVIEW_REQUESTED` ;
- `REGULATORY_REPORT_APPROVED` ;
- `REGULATORY_REPORT_REJECTED` ;
- `REGULATORY_REPORT_SUBMITTED` ;
- `REGULATORY_REPORT_CANCELLED`.

Toute consultation, export ou téléchargement d’un rapport sensible doit également être audité au niveau applicatif.

## Critères d’acceptation

- Les champs obligatoires vides sont refusés.
- Un nouveau rapport est créé à l’état `DRAFT`.
- Seul un brouillon peut demander une revue.
- Le préparateur ne peut pas approuver ou rejeter son rapport.
- Seul un rapport en revue peut être approuvé ou rejeté.
- Seul un rapport approuvé peut être marqué soumis.
- Une référence externe non vide est obligatoire pour la soumission.
- Les états finaux refusent toute nouvelle transition.
- Les tests couvrent création, revue, approbation, soumission, annulation et erreurs principales.
- Le package sécurité expose publiquement le contrat.
