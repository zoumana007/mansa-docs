# 38 — Santé non sensible de la configuration de quarantaine

## Objet

Cette tranche étend la supervision interne de la quarantaine de réconciliation avec un état de santé de configuration volontairement minimal et à faible cardinalité.

L’objectif est de distinguer clairement trois situations opérationnelles sans exposer d’identifiant fournisseur ni de détail de politique :

- aucune politique configurée ;
- des politiques existent mais aucune n’est exploitable par un chemin métier qui exige une politique approuvée ;
- au moins une politique approuvée est disponible.

Cette tranche ne modifie ni les règles de persistance, ni les décisions de quarantaine, ni les garanties fail-closed existantes.

## Endpoint interne

```text
GET /v1/internal/reconciliation/quarantine-policies/health
```

La route reste strictement interne et est protégée par :

1. `WorkloadIdentityGuard` ;
2. `WorkloadScopeGuard` ;
3. le scope explicite `reconciliation:read`.

## Contrat de réponse

La réponse ne contient que :

```json
{
  "data": {
    "status": "EMPTY | NOT_READY | READY",
    "configured": true,
    "ready": true,
    "policyCount": 1,
    "approvedPolicyCount": 1
  }
}
```

Les champs ont les significations suivantes :

- `status = EMPTY` : aucune politique n’est enregistrée ;
- `status = NOT_READY` : au moins une politique est enregistrée mais aucune n’a le statut `APPROVED` ;
- `status = READY` : au moins une politique approuvée est disponible ;
- `configured` indique uniquement si le registre est vide ou non ;
- `ready` indique uniquement si au moins une politique approuvée existe ;
- `policyCount` est le nombre total de politiques ;
- `approvedPolicyCount` est le nombre de politiques approuvées.

## Données explicitement exclues

L’endpoint ne doit jamais exposer :

- `providerId` ;
- rôle autorisé ;
- durée de rétention ;
- classification ;
- secret ;
- payload brut ;
- référence bancaire ;
- identifiant de fichier source ;
- métadonnée fournisseur libre ;
- détail permettant de déduire quel fournisseur est prêt ou non.

Le statut est volontairement global. Il ne constitue pas une API d’inventaire fournisseur.

## Sémantique de readiness

Le statut `READY` signifie uniquement qu’au moins une politique approuvée existe dans le registre.

Il ne garantit pas à lui seul :

- la disponibilité d’un stockage chiffré ;
- la disponibilité d’un fournisseur externe ;
- l’existence d’un adaptateur réel ;
- la capacité à persister un payload brut ;
- la santé d’une base PostgreSQL ;
- le succès d’une opération d’ingestion donnée.

Les chemins métier continuent à utiliser les contrôles déjà définis, notamment la résolution de politique et la politique de persistance de quarantaine.

## Comportement fail-closed conservé

Une configuration `EMPTY` ou `NOT_READY` ne doit pas être transformée en autorisation implicite.

Un fournisseur sans politique approuvée reste refusé lorsque le chemin métier exige `resolve()`.

L’endpoint de santé est purement informatif et ne déclenche :

- aucune mutation ;
- aucune persistance ;
- aucun replay ;
- aucune activation de fournisseur ;
- aucun changement de statut.

## Implémentation

`ReconciliationQuarantinePolicyRegistry` expose désormais une méthode :

```text
health()
```

Elle calcule uniquement les compteurs nécessaires à partir du registre en mémoire et retourne un objet immuable.

`ReconciliationQuarantinePolicyController` expose la méthode :

```text
getConfigurationHealth()
```

sur la route interne décrite ci-dessus.

## Tests

Les tests du contrôleur couvrent maintenant :

- registre vide → `EMPTY` ;
- politiques présentes sans politique approuvée → `NOT_READY` ;
- au moins une politique approuvée → `READY` ;
- absence de `providerId` dans la réponse ;
- immutabilité de la réponse de santé ;
- présence du scope `reconciliation:read` sur le nouvel endpoint ;
- conservation des gardes workload déjà verrouillés par la tranche précédente.

## Cohérence avec les tranches précédentes

Cette tranche complète directement :

- `35-endpoint-interne-inventaire-politiques-quarantaine.md` ;
- `36-resume-operationnel-politiques-quarantaine.md` ;
- `37-contrat-securite-endpoints-quarantaine.md`.

Le document 37 recommandait explicitement un contrat de santé/configuration non sensible permettant de distinguer une configuration vide d’une configuration présente mais non exploitable. Cet écart est désormais couvert.

## Validation attendue

Le dépôt `mansa-platform` doit continuer à passer :

- formatage ;
- lint ;
- TypeScript/build ;
- tests Node de l’API Gateway ;
- tests d’intégration PostgreSQL lorsque l’environnement dédié est activé ;
- contrôles CI existants.

## Prochaine tranche recommandée

Ajouter un indicateur de cohérence de configuration distinct de la simple présence d’une politique uniquement si une invariant supplémentaire apparaît réellement dans le registre. Ne pas enrichir l’endpoint de santé avec des détails fournisseur ; pour toute investigation détaillée, conserver l’inventaire interne protégé existant.