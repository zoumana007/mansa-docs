# Évaluation du risque des opérations

## Objectif

Mansa applique une évaluation de risque déterministe avant les opérations sensibles afin de choisir entre autorisation directe, authentification renforcée, revue manuelle ou blocage.

## Signaux minimaux

L’évaluation peut recevoir :

- le montant en unité monétaire mineure ;
- le pays et le canal d’origine ;
- le niveau d’assurance de la session ;
- la nouveauté de l’appareil ;
- l’écart géographique ou comportemental ;
- la vélocité récente des opérations ;
- la présence d’un bénéficiaire nouveau ;
- les alertes de conformité ou de fraude déjà ouvertes.

Les signaux sont des données dérivées. Aucun secret, jeton, PIN ou donnée biométrique brute ne doit être transmis au moteur de décision.

## Score et niveaux

Le score est un entier borné entre `0` et `100`.

- `LOW` : 0 à 24 ;
- `MEDIUM` : 25 à 49 ;
- `HIGH` : 50 à 74 ;
- `CRITICAL` : 75 à 100.

La politique de score est versionnée. Une modification des poids ne réécrit jamais les décisions historiques.

## Décisions

- `ALLOW` : l’opération peut continuer ;
- `STEP_UP` : une authentification forte récente est exigée ;
- `REVIEW` : l’opération est mise en attente pour revue ;
- `BLOCK` : l’opération est refusée immédiatement.

Une alerte de conformité active ou un score critique conduit au blocage. Un score élevé conduit à la revue. Un score moyen exige une élévation d’authentification sauf si la session est déjà forte.

## Contrat technique partagé

Le paquet `packages/security` expose une fonction pure `evaluateRisk`. Elle :

1. valide que les nombres sont des entiers non négatifs ;
2. additionne les contributions des signaux avec saturation à 100 ;
3. retourne le score, le niveau, la décision et les raisons déclenchées ;
4. produit toujours le même résultat pour les mêmes entrées ;
5. ne réalise aucun appel réseau ni accès à une base de données.

## Traçabilité

Chaque décision persistée contient l’identifiant de l’opération, la version de politique, le score, le niveau, la décision, les raisons, l’horodatage et l’identifiant de corrélation. Les signaux sensibles sont minimisés et soumis aux règles de rétention.

## Critères d’acceptation

1. Le score reste compris entre 0 et 100.
2. Une alerte active produit `BLOCK`.
3. Un score critique produit `BLOCK`.
4. Un score élevé produit `REVIEW`.
5. Un score moyen produit `STEP_UP` lorsque le niveau d’assurance est insuffisant.
6. Les entrées invalides sont refusées.
7. Les tests couvrent les seuils et la saturation.
