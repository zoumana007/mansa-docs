# Filtrage conformité

## Objectif

Le filtrage conformité évalue les correspondances remontées par les fournisseurs de sanctions, de personnes politiquement exposées, de listes internes et de médias défavorables. Il produit une décision pure et traçable avant qu’un dossier KYC, une création de bénéficiaire ou une opération sensible ne soit poursuivi.

Le contrat technique de référence se trouve dans `mansa-platform/packages/security/src/screening.ts`.

## Entités et listes couvertes

Le socle distingue les personnes et les organisations. Les correspondances peuvent provenir de quatre familles :

- `SANCTIONS` : listes de sanctions applicables ;
- `PEP` : personnes politiquement exposées et personnes liées ;
- `INTERNAL_BLOCKLIST` : interdictions internes validées par la conformité ;
- `ADVERSE_MEDIA` : signaux issus de sources médiatiques défavorables.

Chaque correspondance possède un identifiant stable, un score compris entre 0 et 100, une référence de source et un état actif.

## Décisions

- `CLEAR` : aucune correspondance active ne dépasse le seuil de revue.
- `REVIEW` : au moins une correspondance active atteint le seuil de revue sans déclencher de blocage.
- `BLOCK` : une sanction, une entrée de liste interne ou une correspondance dépassant le seuil de blocage est présente.

Une sanction ou une entrée de liste interne active bloque indépendamment de son score. Cette règle empêche qu’un paramétrage de score insuffisant autorise accidentellement une entité explicitement interdite.

## Règles métier

1. Les seuils sont compris entre 0 et 100.
2. Le seuil de blocage est supérieur ou égal au seuil de revue.
3. Les correspondances inactives sont ignorées mais restent conservées pour l’historique.
4. Les décisions retournent uniquement des identifiants de correspondance, jamais les données personnelles complètes.
5. Le moteur de domaine ne contacte aucun fournisseur externe et ne persiste aucune donnée.
6. Les adaptateurs normalisent les réponses des fournisseurs avant l’évaluation.
7. Toute décision `REVIEW` ou `BLOCK` crée un dossier conformité corrélé à l’entité et à l’opération.
8. Les listes, versions, dates de synchronisation et références de source sont auditables.
9. Une décision humaine doit enregistrer l’auteur, la justification, les pièces consultées et la durée de validité.
10. Les données sensibles sont chiffrées et accessibles uniquement aux rôles autorisés.

## Ordre d’intégration

Pour un contrôle KYC ou transactionnel, le service applicatif :

1. collecte les attributs strictement nécessaires ;
2. interroge les adaptateurs de filtrage configurés ;
3. normalise et déduplique les correspondances ;
4. appelle `evaluateScreening` ;
5. poursuit en cas de `CLEAR` ;
6. suspend le traitement et ouvre une revue en cas de `REVIEW` ;
7. bloque l’opération et alerte la conformité en cas de `BLOCK` ;
8. journalise la décision et les références de version des listes.

## Administration

L’administration doit permettre de configurer les fournisseurs, seuils, pays, produits et types d’opération sans exposer de secret. Les clés d’API restent dans le gestionnaire de secrets. Toute modification de seuil en Production est auditée et peut nécessiter une double validation.

Les écrans conformité doivent séparer les données d’identité, les correspondances fournisseur, la décision automatique et la décision humaine. Un agent support général ne doit pas voir les détails sensibles.

## Critères d’acceptation

- Une absence de correspondance pertinente retourne `CLEAR`.
- Une correspondance intermédiaire retourne `REVIEW` avec les identifiants concernés.
- Une sanction ou une liste interne active retourne `BLOCK` quel que soit son score.
- Une correspondance dépassant le seuil de blocage retourne `BLOCK`.
- Les seuils invalides ou incohérents sont rejetés.
- Les correspondances inactives n’influencent pas la décision.
- Les tests unitaires couvrent les trois décisions et la validation de configuration.
- Aucun secret, document KYC réel ou donnée personnelle réelle n’est stocké dans les dépôts.
