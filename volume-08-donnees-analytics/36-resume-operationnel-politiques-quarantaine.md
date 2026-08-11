# 36 — Résumé opérationnel agrégé des politiques de quarantaine

## Objet

Cette tranche ajoute une vue de supervision à cardinalité bornée sur le registre des politiques de quarantaine fournisseurs.

L’objectif est de permettre aux outils internes de connaître l’état global de la configuration sans devoir consommer l’inventaire détaillé et sans exposer d’identifiant fournisseur.

## Endpoint

```text
GET /v1/internal/reconciliation/quarantine-policies/summary
```

La route est portée par `ReconciliationQuarantinePolicyController` et conserve les mêmes contrôles que l’inventaire détaillé :

- `WorkloadIdentityGuard` ;
- `WorkloadScopeGuard` ;
- scope `reconciliation:read`.

Elle reste strictement interne.

## Réponse

La réponse ne contient que des compteurs sur des ensembles fermés :

```json
{
  "data": {
    "total": 4,
    "byStatus": {
      "DRAFT": 1,
      "APPROVED": 2,
      "SUSPENDED": 1
    },
    "byMode": {
      "SIGNALS_ONLY": 3,
      "RAW_SOURCE": 1
    }
  }
}
```

Lorsque le registre est vide, tous les compteurs sont présents avec la valeur `0`.

## Garanties de cardinalité et confidentialité

Le résumé est volontairement borné par les enums internes :

### Statuts

- `DRAFT` ;
- `APPROVED` ;
- `SUSPENDED`.

### Modes

- `SIGNALS_ONLY` ;
- `RAW_SOURCE`.

Il n’expose pas :

- `providerId` ;
- nom de fournisseur ;
- identifiant bancaire ;
- secret ;
- credential ;
- payload brut ;
- rôle autorisé ;
- durée de rétention individuelle ;
- métadonnée permettant de reconstruire une politique fournisseur précise.

Le résultat peut donc alimenter une supervision globale sans créer une métrique à haute cardinalité par fournisseur.

## Implémentation

`ReconciliationQuarantinePolicyRegistry.summary()` :

1. initialise tous les compteurs à zéro ;
2. parcourt les politiques déjà enregistrées en mémoire ;
3. incrémente le compteur correspondant au `status` ;
4. incrémente le compteur correspondant au `mode` ;
5. retourne un objet figé ;
6. fige également les sous-objets `byStatus` et `byMode`.

Le calcul ne modifie aucune politique et ne passe pas par `resolve()`, car le résumé doit aussi refléter les politiques `DRAFT` et `SUSPENDED` à des fins de contrôle de configuration.

La présence d’une politique dans les agrégats ne constitue jamais une autorisation de persister des données brutes. Les décisions métier continuent à utiliser les contrôles fail-closed existants.

## Tests

Le test `apps/api-gateway/test/reconciliation-quarantine-policy.controller.test.mjs` couvre désormais :

- un mélange de politiques `DRAFT`, `APPROVED` et `SUSPENDED` ;
- les modes `SIGNALS_ONLY` et `RAW_SOURCE` ;
- le total global ;
- la présence explicite des compteurs à zéro lorsque le registre est vide ;
- l’immuabilité du résumé et de ses sous-objets ;
- l’absence des identifiants fournisseurs dans la réponse sérialisée.

## Cohérence avec les tranches précédentes

La chaîne de contrôle interne devient :

1. registre provider-neutral ;
2. validation de cohérence ;
3. résolution fail-closed pour les décisions ;
4. garde de persistance ;
5. snapshot immuable détaillé ;
6. endpoint interne d’inventaire ;
7. résumé opérationnel agrégé à cardinalité bornée.

Le résumé ne remplace pas l’inventaire détaillé :

- l’inventaire sert à l’audit de configuration par fournisseur ;
- le résumé sert à la supervision globale et aux tableaux de bord.

## Prochaine tranche recommandée

Ajouter des tests de contrat sur les décorateurs de sécurité de l’endpoint de résumé afin de verrouiller explicitement la présence du scope `reconciliation:read` et des deux gardes internes, sans introduire de dépendance à un fournisseur externe ni exposer de donnée supplémentaire.
