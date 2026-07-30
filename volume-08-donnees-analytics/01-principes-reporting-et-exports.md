# Volume 8 — Reporting, analytics et exports

## 1. Séparation des usages

La base transactionnelle sert aux opérations financières et ne doit pas être utilisée pour des requêtes analytiques lourdes. Les événements validés alimentent progressivement une couche de lecture ou un entrepôt analytique séparé.

## 2. Sources de vérité

- Le grand livre est la source de vérité des mouvements financiers.
- Les modules métier sont la source de vérité de l’état opérationnel.
- Les tableaux de bord sont des vues dérivées et peuvent avoir un léger retard documenté.
- Aucun indicateur dérivé ne peut remplacer une preuve comptable ou un reçu signé.

## 3. Dimensions communes

Les mesures peuvent être segmentées par période, pays, devise, produit, canal, partenaire, commerçant, agence et statut. Les dimensions sensibles sont accessibles uniquement aux rôles autorisés et les petits groupes sont masqués lorsque cela réduit un risque de réidentification.

## 4. Indicateurs prioritaires

- utilisateurs actifs et nouveaux utilisateurs ;
- volume et valeur des transactions ;
- taux de réussite et motifs d’échec ;
- revenus, commissions et coûts partenaires ;
- soldes techniques et écarts de rapprochement ;
- activité commerçant et TPE ;
- délais KYC, support et traitement administratif ;
- fraude, contestations, remboursements et rétrofacturations ;
- disponibilité et latence des services.

## 5. Montants

Les montants restent en unités mineures entières et sont toujours associés à une devise. Les agrégations multidevises ne sont affichées dans une devise commune que si la source, la date et le taux de conversion sont conservés.

## 6. Exports

Les exports volumineux sont générés de façon asynchrone. Une demande possède un statut, un demandeur, des filtres, un format et une date d’expiration. Le téléchargement utilise une référence temporaire, jamais un chemin public permanent.

Formats prévus : CSV, XLSX, PDF et JSON. Les exports réglementaires peuvent imposer un format et une signature spécifiques.

## 7. Sécurité

- Autorisation vérifiée au moment de la demande et du téléchargement.
- Masquage des données selon le rôle.
- Journal d’audit pour création, téléchargement et suppression.
- Chiffrement au repos et en transit.
- Durée de conservation limitée.
- Protection contre les injections de formule dans CSV/XLSX.

## 8. Qualité des données

Chaque pipeline contrôle complétude, unicité, fraîcheur, formats, devises et cohérence des totaux. Les écarts déclenchent une alerte et empêchent, lorsque nécessaire, la publication d’un rapport officiel.

## 9. Rapprochement

Les données Mansa sont comparées aux fichiers ou API des partenaires. Les écarts sont classés, assignés et résolus par correction externe ou écriture compensatoire autorisée. Une transaction historique finalisée n’est jamais modifiée silencieusement.

## 10. Critères d’acceptation

- Les chiffres d’un tableau peuvent être reliés à leurs sources.
- La période et le fuseau horaire sont explicites.
- Les calculs sont reproductibles.
- Les exports expirent et respectent les permissions.
- Les agrégations ne dégradent pas le traitement transactionnel.
