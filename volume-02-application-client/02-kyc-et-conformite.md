# Volume 2 — KYC et conformité client

## 1. Objectif

Le parcours KYC permet d’identifier le client, d’évaluer son niveau de risque et d’activer progressivement les fonctions financières autorisées. Il doit rester configurable par pays, partenaire bancaire et type de compte.

## 2. Niveaux de vérification

- `UNVERIFIED` : compte créé, aucune opération financière sensible autorisée.
- `BASIC` : téléphone ou e-mail vérifié, limites très réduites.
- `STANDARD` : identité et document principal validés, paiements et transferts standards.
- `ENHANCED` : contrôles renforcés, justificatif d’adresse ou origine des fonds selon les règles applicables.
- `REJECTED` : dossier refusé, motif interne conservé et message utilisateur contrôlé.
- `SUSPENDED` : vérification précédemment acceptée mais temporairement bloquée.

Les noms visibles par le client sont localisables. Les codes internes restent stables pour les API et les audits.

## 3. Données minimales

- identité légale ;
- date et pays de naissance ;
- nationalité ;
- adresse et pays de résidence ;
- type, numéro, pays émetteur et dates du document ;
- images du document et preuve de présence lorsque requise ;
- profession, secteur et source de revenus lorsque les règles l’exigent ;
- consentements et versions des politiques acceptées.

Aucune image brute ni donnée biométrique sensible ne doit être exposée dans les journaux applicatifs.

## 4. Cycle de vie du dossier

1. Création d’un dossier brouillon.
2. Collecte et validation locale des champs.
3. Téléversement sécurisé des justificatifs.
4. Soumission avec identifiant d’idempotence.
5. Contrôles automatiques et, si nécessaire, revue humaine.
6. Décision : approbation, demande de complément ou rejet.
7. Mise à jour du niveau KYC, des limites et des fonctions autorisées.
8. Audit de chaque transition et notification contrôlée du client.

## 5. Règles fonctionnelles

- Un client ne peut avoir qu’un dossier actif par programme KYC.
- Toute nouvelle soumission crée une version du dossier ; elle ne réécrit pas silencieusement l’historique.
- Les motifs internes de risque sont séparés des messages affichés au client.
- Toute décision manuelle exige l’identité de l’agent, un motif et un horodatage.
- Un document expiré déclenche une révision selon les règles du pays.
- Les limites financières sont déterminées par une politique, jamais codées directement dans l’application mobile.

## 6. Contrôles de risque

Le système prévoit des adaptateurs pour : vérification documentaire, détection de présence, listes de sanctions, personnes politiquement exposées, fraude documentaire et vérification de numéro. Les fournisseurs restent remplaçables et leurs réponses brutes sont conservées dans un stockage protégé avec durée de conservation définie.

## 7. Expérience utilisateur

- progression claire par étapes ;
- possibilité de reprendre un brouillon ;
- explication précise des documents acceptés ;
- contrôle de qualité avant envoi ;
- statut visible sans exposer les règles internes ;
- accessibilité, faible consommation de données et reprise après coupure réseau.

## 8. Critères d’acceptation

- Les statuts et niveaux sont partagés entre documentation, API et applications.
- Une soumission répétée avec la même clé d’idempotence ne crée pas de doublon.
- Les pièces sont chiffrées et accessibles uniquement aux rôles autorisés.
- Chaque décision est auditée.
- Le changement de niveau applique immédiatement les limites correspondantes.
- Les erreurs externes sont traduites en états métier stables et réessayables lorsque pertinent.
