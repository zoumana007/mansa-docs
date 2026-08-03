# Surveillance transactionnelle

## Objectif

La surveillance transactionnelle agrège des signaux de risque produits par les règles métier, les profils clients, les historiques d’activité et les adaptateurs conformité. Elle fournit une décision déterministe avant l’autorisation définitive d’une opération ou pendant une analyse post-transaction.

Le contrat technique de référence se trouve dans `mansa-platform/packages/security/src/transaction-monitoring.ts`.

## Signaux couverts

Le premier socle normalise les familles suivantes :

- `STRUCTURING` : fractionnement présumé d’opérations pour contourner un seuil ;
- `RAPID_MOVEMENT` : entrée puis sortie rapide de fonds ;
- `UNUSUAL_AMOUNT` : montant inhabituel par rapport au profil attendu ;
- `HIGH_RISK_COUNTRY` : exposition à un pays classé à risque selon la politique active ;
- `DORMANT_ACCOUNT_ACTIVITY` : reprise soudaine d’activité sur un compte dormant ;
- `MANY_BENEFICIARIES` : multiplication inhabituelle des bénéficiaires.

Chaque signal contient un identifiant stable, un score de 0 à 100, une référence de règle ou de source et un état actif.

## Décisions

- `ALLOW` : aucun signal actif n’atteint le seuil de revue et le cumul reste inférieur à ce seuil.
- `REVIEW` : un signal ou le score cumulé atteint le seuil de revue sans atteindre le seuil de blocage.
- `BLOCK` : un signal obligatoire, un signal individuel ou le cumul atteint le seuil de blocage.

Le score cumulé est plafonné à 100. Le moteur retourne les identifiants des signaux ayant justifié la décision sans exposer les données personnelles ou les paramètres secrets des règles.

## Règles métier

1. Les seuils de revue et de blocage sont compris entre 0 et 100.
2. Le seuil de blocage est supérieur ou égal au seuil de revue.
3. Les signaux inactifs sont ignorés par la décision mais conservés dans l’historique.
4. Certaines familles peuvent être configurées comme bloquantes indépendamment de leur score.
5. Un cumul de signaux faibles peut provoquer une revue ou un blocage.
6. La fonction de domaine ne lit pas la base, ne contacte aucun fournisseur et ne modifie aucune transaction.
7. Les règles de détection, leurs versions et leurs périodes d’application sont auditables.
8. Une décision `REVIEW` ouvre un dossier conformité avec priorité, propriétaire et délai de traitement.
9. Une décision `BLOCK` doit produire un motif client générique et un motif interne détaillé.
10. Les décisions manuelles exigent une justification et ne suppriment jamais le résultat automatique initial.

## Chaîne d’intégration

Le service applicatif :

1. reçoit une transaction normalisée et son contexte ;
2. calcule ou collecte les signaux applicables ;
3. élimine les doublons et signaux expirés ;
4. appelle `evaluateTransactionMonitoring` ;
5. poursuit l’opération en cas de `ALLOW` ;
6. suspend ou limite l’opération en cas de `REVIEW` selon le produit ;
7. bloque l’opération en cas de `BLOCK` ;
8. journalise la décision, les identifiants des signaux et les versions de règles ;
9. alimente le dossier conformité et les métriques opérationnelles.

## Administration

Les seuils, signaux obligatoires, produits, canaux, pays et profils concernés sont configurables par environnement. Toute modification en Production est auditée et peut nécessiter une double validation. Les règles ne doivent contenir ni clé d’API, ni identifiant partenaire secret, ni donnée personnelle réelle.

Les écrans d’administration distinguent la configuration des règles, les alertes générées, les dossiers en cours, les décisions humaines et les statistiques. Les agents support non habilités ne voient pas les détails des signaux conformité.

## Critères d’acceptation

- Une opération sans signal pertinent retourne `ALLOW`.
- Plusieurs signaux faibles peuvent déclencher `REVIEW` par cumul.
- Un signal configuré comme obligatoire déclenche `BLOCK` quel que soit son score.
- Un signal ou cumul atteignant le seuil de blocage déclenche `BLOCK`.
- Les signaux inactifs n’influencent pas la décision.
- Les seuils invalides ou incohérents sont rejetés.
- Les tests unitaires couvrent autorisation, cumul, blocage obligatoire et validation.
- Aucun secret ni donnée personnelle réelle n’est stocké dans les dépôts.
