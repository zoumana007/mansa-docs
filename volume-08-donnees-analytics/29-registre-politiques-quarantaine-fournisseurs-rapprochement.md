# 29 — Registre de politiques de quarantaine par fournisseur pour le rapprochement

## Objet

Cette tranche prolonge la politique durable de quarantaine définie dans `28-politique-durable-quarantaine-ingestion-rapprochement.md`.

Elle ajoute un registre explicite, provider-neutral et fail-closed permettant de représenter les décisions de sécurité applicables à chaque fournisseur de données de rapprochement, sans stocker de secret et sans activer à lui seul la persistance de données rejetées.

## État implémenté

Dans `mansa-platform`, `ReconciliationQuarantinePolicyRegistry` permet désormais d’enregistrer et de résoudre des politiques par `providerId`.

Le registre représente :

- la classification de données : `INTERNAL`, `CONFIDENTIAL` ou `RESTRICTED` ;
- le mode demandé : `SIGNALS_ONLY` ou `RAW_SOURCE` ;
- la durée de rétention lorsqu’un stockage brut est explicitement approuvé ;
- l’exigence de chiffrement au repos ;
- l’exigence de chiffrement en transit ;
- les rôles habilités ;
- le statut de replay : `DISABLED` ou `MANUAL_REVIEW` ;
- le statut de politique : `DRAFT`, `APPROVED` ou `SUSPENDED`.

Le registre est enregistré comme provider NestJS dans `ReconciliationModule` et exporté pour les futurs composants du pipeline.

## Fail-closed

Le comportement par défaut reste fermé :

```text
fournisseur sans politique
→ aucune résolution
→ erreur explicite
→ aucune autorisation implicite de stockage
```

Une politique enregistrée mais non approuvée ne peut pas être résolue comme politique active.

## Règles pour SIGNALS_ONLY

Une politique `SIGNALS_ONLY` :

- ne définit aucune durée de rétention ;
- conserve le replay désactivé ;
- peut être enregistrée en brouillon ou suspendue mais ne devient active qu’en statut `APPROVED` ;
- n’autorise toujours aucun stockage brut du fichier fournisseur.

Cette règle reste cohérente avec la politique précédente, où seules les métriques bornées et non identifiantes sont autorisées.

## Règles pour RAW_SOURCE

Le registre peut représenter une décision future `RAW_SOURCE`, mais uniquement si la politique est cohérente et explicitement approuvée.

Les contrôles minimaux sont :

- statut `APPROVED` ;
- `retentionDays` entier strictement positif ;
- chiffrement au repos obligatoire ;
- chiffrement en transit obligatoire ;
- au moins un rôle habilité ;
- aucun rôle vide.

Cette représentation **n’active pas le stockage brut**.

`ReconciliationQuarantinePersistencePolicy` continue à bloquer `RAW_SOURCE` tant qu’une tranche ultérieure n’a pas implémenté le stockage, le chiffrement effectif, l’audit d’accès, la suppression vérifiable et les contrôles de replay.

La combinaison attendue reste donc :

```text
registre fournisseur approuvé
+ politique globale de persistance
+ implémentation de stockage conforme
+ contrôles d’accès et audit
= condition nécessaire avant toute conservation brute
```

Aucun de ces éléments ne doit être contourné.

## Immutabilité

Une politique résolue est figée, ainsi que sa liste de rôles.

Cela évite qu’un composant consommateur modifie silencieusement une décision de sécurité après son enregistrement.

Toute évolution future d’une politique devra passer par un mécanisme explicite de versionnement ou de remplacement contrôlé, et non par mutation en mémoire.

## Doublons

Le registre refuse l’enregistrement d’une deuxième politique pour le même `providerId` dans une même instance.

Cette décision évite qu’un ordre d’initialisation ou une dépendance NestJS modifie implicitement la politique active.

Une future fonction de mise à jour devra être volontaire, auditée et versionnée.

## Secrets

Le registre ne doit contenir :

- aucune clé de chiffrement ;
- aucun token fournisseur ;
- aucun mot de passe ;
- aucun secret HMAC ;
- aucune URL signée ;
- aucun payload de rapprochement ;
- aucune ligne rejetée.

Les clés et secrets restent dans l’infrastructure de secrets/KMS/HSM prévue par la plateforme.

## Tests ajoutés

Les tests de `mansa-platform` couvrent :

- résolution d’une politique explicitement approuvée ;
- normalisation du `providerId` ;
- immutabilité de la politique et des rôles ;
- fail-closed pour fournisseur inconnu ;
- refus de résolution d’une politique brouillon ;
- interdiction de rétention en mode `SIGNALS_ONLY` ;
- interdiction de replay en mode `SIGNALS_ONLY` ;
- obligation d’approbation pour `RAW_SOURCE` ;
- obligation d’une rétention positive ;
- obligation du chiffrement au repos et en transit ;
- obligation d’au moins un rôle habilité ;
- refus des doublons.

## Invariant de sécurité

La présence d’une politique `RAW_SOURCE` dans le registre ne doit jamais être interprétée comme une autorisation suffisante de persister une source rejetée.

Le registre décrit une décision approuvée ; il ne remplace pas les contrôles techniques de persistance.

## Prochaine tranche recommandée

Brancher ce registre sur la frontière d’ingestion afin que chaque source rejetée soit évaluée avec une politique fournisseur explicite, tout en conservant le comportement actuel `SIGNALS_ONLY` pour les fournisseurs sans décision durable approuvée. La tranche suivante devra surtout vérifier que l’intégration du registre ne crée aucun stockage brut implicite et ne dégrade pas les métriques de quarantaine existantes.
