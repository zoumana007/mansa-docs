# 34 — Inventaire en lecture seule des politiques de quarantaine

## Objet

Cette tranche ajoute une vue opérationnelle sûre du registre des politiques de quarantaine fournisseurs. Elle fait suite au contrôle d’assemblage du module et prépare l’audit, la supervision et les futures interfaces d’administration sans exposer de payload fournisseur ni modifier une décision de persistance.

## État implémenté

Dans `mansa-platform`, `ReconciliationQuarantinePolicyRegistry` expose désormais :

```ts
snapshot(): readonly ReconciliationQuarantineProviderPolicy[]
```

Cette méthode retourne un inventaire des politiques enregistrées.

Le snapshot :

- est trié par `providerId` afin d’être déterministe ;
- est figé avec `Object.freeze` ;
- contient des copies figées des politiques ;
- conserve `allowedRoles` sous forme de tableau figé ;
- inclut les politiques `DRAFT`, `APPROVED` et `SUSPENDED` déjà présentes dans le registre ;
- n’appelle pas `resolve()` et n’accorde donc aucun droit supplémentaire ;
- ne déclenche aucune persistance ;
- ne stocke ni ne retourne de payload fournisseur.

## Pourquoi inclure les politiques non approuvées

Une vue d’audit doit pouvoir montrer qu’une politique existe même si elle n’est pas exploitable pour une décision active.

Le comportement de décision reste séparé :

- `resolve()` refuse une politique non `APPROVED` ;
- `snapshot()` inventorie l’état enregistré sans l’interpréter comme une autorisation.

Cette séparation évite de confondre visibilité opérationnelle et activation fonctionnelle.

## Déterminisme

L’ordre d’enregistrement dans le processus ne doit pas modifier la sortie d’audit.

Le registre trie donc les politiques par `providerId`. Deux instances contenant les mêmes politiques doivent produire le même ordre logique, indépendamment de leur ordre d’insertion.

## Immutabilité

Le snapshot est détaché de la collection interne du registre.

Une nouvelle politique enregistrée après création d’un snapshot ne modifie pas le snapshot déjà obtenu. Les consommateurs peuvent donc l’utiliser comme photographie ponctuelle pour :

- audit ;
- diagnostic ;
- métriques ;
- export administratif futur ;
- comparaison de configuration.

Aucun consommateur ne peut muter le registre en modifiant le tableau ou `allowedRoles` dans le snapshot.

## Tests ajoutés

`apps/api-gateway/test/reconciliation-quarantine-policy-registry.test.mjs` couvre désormais :

- tri déterministe par fournisseur ;
- gel du tableau retourné ;
- gel des politiques ;
- gel de `allowedRoles` ;
- impossibilité de pousser un élément dans le snapshot ;
- impossibilité de modifier les rôles d’un élément ;
- indépendance d’un snapshot par rapport aux enregistrements ultérieurs.

## Sécurité

Cette tranche ne change aucune règle fail-closed.

Elle n’active pas `RAW_SOURCE`, ne permet aucun replay manuel et n’introduit aucun secret. Elle expose uniquement les métadonnées de politique déjà présentes en mémoire.

Le principe reste :

```text
visibilité d’une politique
!=
autorisation de stockage brut
```

## Cohérence documentation / code

Le lot complète la chaîne suivante :

1. registre fournisseur ;
2. validation des politiques ;
3. décision fail-closed ;
4. garde globale de persistance ;
5. injection NestJS ;
6. contrôle d’assemblage ;
7. inventaire immuable en lecture seule.

## Prochaine tranche recommandée

Ajouter un résumé opérationnel agrégé du snapshot, sans identifiant sensible supplémentaire, donnant au minimum le nombre de politiques par statut et par mode. Ce résumé devra rester déterministe, borné et indépendant des payloads fournisseurs afin de pouvoir alimenter la supervision sans dupliquer les règles de décision.
