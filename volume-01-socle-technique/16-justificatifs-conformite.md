# Justificatifs de conformité

## Objectif

Les justificatifs de conformité rattachent à un dossier les éléments utilisés par les analystes pour prendre une décision : document protégé, transaction, résultat de filtrage, note interne ou référence externe.

Le contrat technique de référence se trouve dans `mansa-platform/packages/security/src/compliance-evidence.ts`.

## Principes

1. Le dépôt de code ne contient jamais le contenu réel des justificatifs.
2. Le domaine conserve uniquement une référence opaque vers un stockage autorisé.
3. Un checksum peut être enregistré pour vérifier l’intégrité de la pièce.
4. Toute pièce est classifiée `INTERNAL`, `CONFIDENTIAL` ou `RESTRICTED`.
5. L’ajout et le retrait produisent une action d’audit stable.
6. Un retrait est logique : la référence et l’historique restent conservés.
7. Un justificatif déjà retiré ne peut pas être retiré une seconde fois.

## Types supportés

- `DOCUMENT` : pièce KYC, attestation ou document métier stocké dans un coffre documentaire ;
- `TRANSACTION` : référence vers une transaction ou un ensemble d’écritures ;
- `SCREENING_RESULT` : résultat détaillé d’un filtrage sanctions ou PPE ;
- `ANALYST_NOTE` : note d’analyse stockée dans un espace protégé ;
- `EXTERNAL_REFERENCE` : identifiant d’un dossier partenaire ou d’une source externe.

## Données minimales

Chaque justificatif contient :

- un identifiant unique ;
- l’identifiant du dossier conformité ;
- un type ;
- une référence opaque ;
- un checksum optionnel ;
- un niveau de classification ;
- l’acteur et la date d’ajout ;
- en cas de retrait, l’acteur, la date et la justification.

## Stockage

Les documents et notes sensibles doivent être stockés dans un service distinct disposant :

- d’un chiffrement au repos et en transit ;
- d’un contrôle d’accès par rôle et périmètre ;
- d’URL temporaires ou de jetons de consultation à courte durée ;
- d’une journalisation des consultations ;
- d’une politique de rétention conforme aux obligations applicables ;
- d’une procédure de gel légal lorsque nécessaire.

La valeur `reference` ne doit pas être une URL publique permanente. Les secrets d’accès, clés de chiffrement et jetons signés ne sont jamais persistés dans le domaine ni écrits dans les journaux.

## Cycle d’ajout

1. Le service applicatif vérifie que le dossier existe et n’est pas fermé, sauf règle explicite de conservation.
2. Il contrôle la permission, le périmètre et la classification demandée.
3. Il charge ou enregistre la pièce dans le stockage protégé.
4. Il calcule le checksum lorsque cette vérification est requise.
5. Il appelle `addComplianceEvidence` avec une référence opaque.
6. Il persiste le métadonné et l’événement `COMPLIANCE_EVIDENCE_ADDED` atomiquement.

## Cycle de retrait

Un retrait exige une justification non vide. Le service :

1. vérifie que la pièce est encore active ;
2. contrôle l’autorisation renforcée ;
3. appelle `removeComplianceEvidence` ;
4. conserve la référence et les métadonnées historiques ;
5. produit `COMPLIANCE_EVIDENCE_REMOVED` ;
6. applique séparément la politique de suppression ou de rétention du stockage physique.

Le retrait métier ne signifie donc pas automatiquement l’effacement immédiat du fichier physique.

## Autorisations

- Les agents conformité habilités peuvent ajouter des justificatifs dans leur périmètre.
- L’accès à une pièce `RESTRICTED` requiert une permission dédiée et doit être audité.
- Le support général ne doit recevoir ni la référence de stockage ni le contenu sensible.
- Le retrait peut exiger une double validation en Production.
- Un acteur ne peut pas modifier les métadonnées d’auteur ou d’horodatage.

## Critères d’acceptation

- Les champs obligatoires vides sont refusés.
- L’ajout retourne une action `COMPLIANCE_EVIDENCE_ADDED`.
- Le retrait conserve la référence initiale et retourne `COMPLIANCE_EVIDENCE_REMOVED`.
- Le retrait sans justification est refusé.
- Un second retrait est refusé.
- L’état actif est déterminé par l’absence de `removedAt`.
- Les tests unitaires couvrent l’ajout, le retrait logique et les erreurs principales.
- L’export public du package sécurité expose le contrat.
