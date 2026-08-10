# 14 — Observabilité opérationnelle du rapprochement

## Objet

Ce lot ajoute un premier socle d’observabilité locale au moteur de rapprochement financier sans exposer de payload fournisseur ni de donnée sensible.

## État implémenté

`mansa-platform/apps/api-gateway/src/reconciliation/reconciliation-operational-monitor.ts` fournit un composant NestJS dédié qui conserve, pour le processus courant :

- nombre d’imports démarrés ;
- nombre d’imports réussis ;
- nombre d’imports échoués ;
- nombre total d’items importés avec succès ;
- horodatage du dernier démarrage ;
- horodatage du dernier succès ;
- horodatage du dernier échec.

Le service d’import de rapprochement enregistre désormais ces événements autour de l’opération de préparation et de persistance. Une exception du provider adapter ou du repository incrémente le compteur d’échec puis est relancée sans être masquée.

Le monitor est enregistré et exporté par `ReconciliationModule` pour permettre à un futur exporter de métriques ou endpoint d’administration interne de le consommer sans coupler le domaine à Prometheus, OpenTelemetry ou un fournisseur SaaS précis.

## Garanties

Le snapshot opérationnel ne doit jamais contenir :

- lignes brutes du fournisseur ;
- références de transactions ;
- identifiants client ;
- métadonnées d’un lot ;
- empreintes de lignes ;
- secrets ou credentials ;
- données KYC ;
- montants individuels.

Les compteurs sont strictement process-local. Ils servent à vérifier le comportement applicatif et à fournir une source minimale à un futur exporter, pas à constituer une source d’audit durable.

## Tests

`apps/api-gateway/test/reconciliation-operational-monitor.test.mjs` couvre :

- l’enregistrement d’un import réussi ;
- l’enregistrement d’un échec ;
- l’indépendance des compteurs succès/échec ;
- le cumul du nombre d’items ;
- le rejet des compteurs invalides.

Le script de test existant de l’API Gateway compile d’abord TypeScript puis exécute les fichiers `test/*.test.mjs`, ce qui inclut automatiquement ce nouveau test.

## Limites explicites

Ce lot ne prétend pas fournir une observabilité distribuée de production. Il manque encore :

1. un exporter de métriques approuvé ;
2. des métriques par résultat métier suffisamment agrégées pour éviter les cardinalités explosives ;
3. des alertes configurables ;
4. des SLI/SLO ;
5. la corrélation traces/logs/métriques ;
6. un dashboard d’exploitation ;
7. la conservation externe des séries temporelles ;
8. des tests d’intégration de l’exporter retenu.

## Prochaine tranche recommandée

Définir un contrat d’export indépendant du fournisseur puis publier des métriques à faible cardinalité, notamment :

- imports réussis/échoués ;
- durée d’import ;
- items rapprochés vs en écart ;
- résolutions opérateur ;
- taille de backlog non résolu ;
- âge du plus ancien écart non résolu.

Les dimensions autorisées doivent rester bornées, par exemple environnement et type de résultat. Les identifiants de tenant, transaction, lot, client ou fichier ne doivent pas devenir des labels de métriques.
