# Contrat métier — Identifiant Mansa

## Statut

- Domaine : application Client / Mansa Connect
- Implémentation de référence : `mansa-platform/packages/domain/src/mansa-handle.ts`
- Niveau : valeur objet du noyau métier

## Objectif

L’identifiant Mansa est un identifiant public permettant de rechercher un utilisateur, ouvrir une conversation et initier des opérations financières sans exposer obligatoirement son numéro de téléphone.

## Normalisation

À l’entrée du domaine :

1. les espaces en début et fin sont supprimés ;
2. le préfixe `@` est accepté puis retiré du stockage interne ;
3. les lettres sont converties en minuscules ;
4. la valeur normalisée est utilisée pour l’unicité et les comparaisons.

La représentation d’affichage réintroduit le préfixe `@`.

## Format par défaut

- Longueur minimale : 3 caractères.
- Longueur maximale : 30 caractères.
- Caractères autorisés : lettres latines minuscules, chiffres, point, tiret bas et tiret.
- Le premier et le dernier caractère doivent être une lettre ou un chiffre.
- Les espaces et autres caractères spéciaux sont refusés.

Les limites et la liste des identifiants réservés sont configurables par politique métier. Une politique incohérente doit être refusée avant la création de l’identifiant.

## Identifiants réservés

Le noyau fournit une liste minimale comprenant notamment les termes liés à Mansa, au support, à l’administration, aux banques, à l’État et aux universités. L’administration devra compléter cette liste par pays et partenaire.

La réservation est évaluée après normalisation. Ainsi `@Support`, `support` et ` SUPPORT ` désignent la même valeur réservée.

## Invariants

- Un identifiant normalisé correspond à une seule valeur métier.
- Deux identifiants équivalents après normalisation sont égaux.
- La valeur créée est immuable.
- La valeur objet ne vérifie pas la disponibilité en base : cette responsabilité appartient au service applicatif et au dépôt persistant.
- La valeur objet ne gère ni l’historique de changement, ni la période de quarantaine, ni les paramètres de découvrabilité.

## Persistance attendue

La base doit imposer une contrainte unique sur la valeur normalisée. La modification d’un identifiant doit être transactionnelle et produire un événement d’audit. Les anciens identifiants doivent être placés en quarantaine pendant une durée configurable avant toute réattribution éventuelle.

## API attendue

Les routes applicatives devront au minimum couvrir :

- vérification de disponibilité ;
- attribution initiale ;
- modification contrôlée ;
- lecture du profil public ;
- recherche selon les paramètres de confidentialité ;
- réservation et libération administratives.

Aucune route ne doit se contenter de la validation syntaxique : les contrôles d’unicité, de fréquence de modification, de statut du compte, de sécurité et d’audit restent obligatoires.

## Critères de recette

1. `  @Camara.007  ` est stocké comme `camara.007` et affiché comme `@camara.007`.
2. `@Zoumana` et `zoumana` sont considérés comme égaux.
3. Un identifiant commençant ou finissant par un séparateur est refusé.
4. Un identifiant réservé est refusé sans distinction de casse.
5. Les politiques de longueur incohérentes sont refusées.
6. Les tests automatisés du package domaine couvrent normalisation, égalité, format, réservation et configuration.
