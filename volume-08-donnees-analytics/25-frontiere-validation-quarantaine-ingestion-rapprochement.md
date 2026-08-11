# 25 — Frontière de validation et quarantaine d’ingestion du rapprochement

## Objectif

Cette tranche formalise une frontière provider-neutral placée avant la résolution d’un adaptateur fournisseur.

Elle doit empêcher qu’une source manifestement invalide ou démesurée atteigne le pipeline de préparation, le repository ou un futur connecteur partenaire.

La première implémentation se trouve dans `ReconciliationIngestionBoundary`.

## Principes

La frontière :

1. ne connaît aucun protocole bancaire réel ;
2. ne dépend d’aucun secret ;
3. ne persiste aucun payload brut ;
4. produit une décision bornée ;
5. utilise une liste fermée de codes de rejet ;
6. peut indiquer l’index d’une ligne fautive sans retourner son contenu ;
7. calcule une empreinte technique de source qui ne contient pas les lignes brutes.

## Décisions possibles

Une source valide retourne :

```text
accepted = true
sourceFingerprint = sha256(...)
rowCount = N
```

Une source refusée retourne :

```text
accepted = false
code = <code fermé>
rowIndex = <optionnel>
sourceFingerprint = sha256(...)
rowCount = N
```

## Codes de quarantaine

La première liste fermée est :

```text
PROVIDER_ID_REQUIRED
INVALID_PERIOD
EMPTY_SOURCE
SOURCE_TOO_LARGE
INVALID_PROVIDER_REFERENCE
INVALID_AMOUNT
INVALID_CURRENCY
INVALID_STATUS
```

Un nouveau code doit être ajouté explicitement au contrat et testé. Les messages libres provenant d’un parser ou d’un partenaire ne doivent pas devenir des catégories opérationnelles non bornées.

## Limite volumétrique

La frontière applique une limite de lignes configurable.

La valeur par défaut du socle est :

```text
100000 lignes
```

Cette valeur est une protection technique initiale, pas une limite produit définitive. Une intégration partenaire réelle devra définir ses propres limites de fichier, streaming, pagination ou chunking à partir du format officiel.

## Validation de période

Le système refuse :

- une date invalide ;
- une fin antérieure au début.

La frontière ne choisit pas elle-même une période par défaut.

## Validation des lignes

Pour chaque ligne fournisseur normalisée, la frontière contrôle au minimum :

- référence fournisseur non vide ;
- montant en entier sûr positif ou nul ;
- devise sous forme de code de trois lettres ;
- statut non vide.

La normalisation métier détaillée reste la responsabilité de l’adaptateur. Cette frontière sert uniquement de filtre générique avant traitement.

## Confidentialité

La décision de quarantaine ne doit jamais contenir :

- ligne brute ;
- nom complet ;
- téléphone ;
- PAN ;
- IBAN/RIB ;
- token ;
- credential ;
- contenu de fichier ;
- métadonnée fournisseur non explicitement autorisée.

L’empreinte de source actuelle est construite à partir de métadonnées bornées : identifiant fournisseur, référence de fichier normalisée, période et nombre de lignes.

Elle n’est pas une preuve cryptographique du contenu complet du fichier et ne remplace pas l’empreinte déterministe calculée ensuite par l’adaptateur sur la source normalisée.

## Quarantaine

Dans cette tranche, « quarantaine » signifie une décision de rejet structurée.

Aucun stockage durable de quarantaine n’est encore introduit afin de ne pas inventer :

- une politique de conservation ;
- un chiffrement spécifique ;
- un backend objet ;
- un workflow opérateur ;
- des permissions de consultation ;
- une durée de rétention.

Un futur stockage de quarantaine devra être explicitement conçu avec minimisation, chiffrement, audit, durée de conservation et purge.

## Tests

La tranche couvre :

- acceptation d’une source bornée ;
- providerId vide ;
- période invalide ;
- source vide ;
- source trop volumineuse ;
- ligne avec montant invalide ;
- absence de ligne brute dans la décision ;
- validation de la configuration `maxRows`.

## Intégration recommandée au pipeline

La prochaine étape est de placer cette frontière dans le service d’import avant :

```text
providerRegistry.resolve(...)
```

Le flux cible devient :

```text
source reçue
→ frontière validation/quarantaine
→ résolution unique de l’adaptateur
→ préparation provider-specific
→ repository
→ monitoring / SLO / alerting
```

Toute décision de quarantaine devra incrémenter un compteur opérationnel à faible cardinalité sans exposer le fournisseur, le tenant ou le contenu brut.

## Hors périmètre

Cette tranche ne définit pas :

- ingestion SFTP ;
- webhook ;
- upload HTTP ;
- parser CSV spécifique ;
- ISO 20022 ;
- PGP ;
- antivirus ;
- stockage objet ;
- file de messages ;
- DLQ ;
- mécanisme de reprise ;
- interface opérateur de quarantaine.

Ces éléments dépendront de l’infrastructure cible et des formats partenaires réels.
