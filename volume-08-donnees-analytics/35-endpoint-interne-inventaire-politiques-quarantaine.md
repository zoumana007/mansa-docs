# 35 — Endpoint interne d’inventaire des politiques de quarantaine

## Objet

Cette tranche rend l’inventaire immuable des politiques de quarantaine consultable par les workloads internes autorisés, sans exposer de payload fournisseur ni modifier les règles de décision ou de persistance.

Elle complète le snapshot en mémoire introduit précédemment par une surface HTTP interne protégée.

## Endpoint

```text
GET /v1/internal/reconciliation/quarantine-policies
```

La route est portée par `ReconciliationQuarantinePolicyController` et protégée par :

- `WorkloadIdentityGuard` ;
- `WorkloadScopeGuard` ;
- le scope `reconciliation:read`.

Elle n’est pas destinée à une exposition publique.

## Réponse

La forme logique est :

```json
{
  "data": [
    {
      "providerId": "provider-example",
      "classification": "CONFIDENTIAL",
      "mode": "SIGNALS_ONLY",
      "retentionDays": null,
      "encryptionAtRestRequired": true,
      "encryptionInTransitRequired": true,
      "allowedRoles": [],
      "replayStatus": "DISABLED",
      "status": "APPROVED"
    }
  ]
}
```

La réponse réutilise directement le snapshot déterministe et immuable du registre.

## Garanties de sécurité

La route :

- n’expose aucun payload brut fournisseur ;
- n’expose aucun secret ;
- ne contient ni PAN, ni clé, ni credential ;
- ne déclenche aucune persistance ;
- ne modifie aucune politique ;
- ne transforme pas une politique `DRAFT` ou `SUSPENDED` en politique exploitable ;
- ne contourne pas `resolve()` pour les décisions métier ;
- reste en lecture seule.

L’inventaire peut inclure des politiques non approuvées afin de permettre l’audit de configuration, mais leur présence dans cette vue n’accorde aucune autorisation fonctionnelle.

## Câblage NestJS

`ReconciliationQuarantinePolicyController` est enregistré dans `ReconciliationModule` avec les contrôleurs de réconciliation existants.

Le registre reste le même singleton NestJS utilisé par les services de décision et de persistance, ce qui évite une vue parallèle ou une duplication de configuration.

## Tests

Le test `apps/api-gateway/test/reconciliation-quarantine-policy.controller.test.mjs` vérifie :

- la restitution déterministe par `providerId` ;
- l’utilisation du snapshot figé ;
- l’absence de champs génériques `payload` ou `secret` ;
- le comportement avec registre vide.

Les gardes et le scope sont déclarés au niveau des décorateurs NestJS ; la validation d’assemblage globale du module doit continuer à vérifier que le contrôleur est bien enregistré dans le graphe de dépendances.

## Cohérence avec le lot précédent

La chaîne opérationnelle devient :

1. enregistrement d’une politique provider-neutral ;
2. validation de cohérence ;
3. résolution fail-closed pour les décisions ;
4. garde de persistance ;
5. snapshot immuable ;
6. lecture HTTP interne protégée.

Aucune étape de cette chaîne ne permet à l’endpoint d’administration de devenir une commande d’activation ou de stockage brut.

## Prochaine tranche recommandée

Ajouter un résumé opérationnel agrégé du registre donnant au minimum les nombres par `status` et par `mode`, sans ajouter d’identifiant sensible et sans exposer les payloads fournisseurs. Ce résumé pourra alimenter la supervision et les tableaux de bord internes avec une cardinalité bornée.
