# 28 — Politique durable de quarantaine d’ingestion du rapprochement

## Objet

Cette tranche formalise la politique de sécurité qui doit précéder tout stockage durable d’une source fournisseur rejetée par le pipeline de rapprochement.

Elle prolonge :

- `25-frontiere-validation-quarantaine-ingestion-rapprochement.md` ;
- `26-integration-quarantaine-pipeline-rapprochement.md` ;
- `27-metriques-quarantaine-ingestion-rapprochement.md`.

L’objectif immédiat n’est pas de créer un bucket, une DLQ ou une table contenant les fichiers rejetés. L’objectif est au contraire d’imposer un comportement **fail-closed** : tant que les contrôles de conservation durable ne sont pas approuvés et implémentés, seules les métriques bornées et non identifiantes restent autorisées.

## État implémenté

Dans `mansa-platform`, `ReconciliationQuarantinePersistencePolicy` matérialise cette décision :

- le mode par défaut et seul mode accepté est `SIGNALS_ONLY` ;
- `persistRawSource = false` ;
- `persistProviderPayload = false` ;
- `durableMetadataAllowed = false` ;
- `manualReplayAllowed = false` ;
- toute tentative d’activer `RAW_SOURCE` échoue explicitement ;
- une garde dédiée refuse également toute tentative de stockage brut sans politique future approuvée.

Cette tranche n’ajoute donc aucun stockage de données rejetées.

## Pourquoi le stockage brut reste interdit

Une source de rapprochement fournisseur peut contenir ou permettre d’inférer :

- références de transactions ;
- montants ;
- devise ;
- statut de paiement ;
- identifiants techniques du fournisseur ;
- périodes d’activité ;
- données opérationnelles sensibles ;
- éventuellement des données personnelles selon le format partenaire.

Conserver automatiquement une source invalide sans cadre défini augmenterait inutilement :

- la surface d’exposition ;
- la durée de conservation ;
- le risque d’accès interne excessif ;
- le risque de fuite dans les logs ou outils de support ;
- la complexité des demandes de suppression ;
- le risque de rejouer un fichier non fiable.

## Conditions minimales avant un futur mode durable

Avant qu’un mode de stockage de quarantaine puisse être ajouté, les éléments suivants doivent être décidés puis testés.

### Classification

Définir précisément la classe de données contenue dans les sources de chaque fournisseur et interdire toute hypothèse implicite selon laquelle un fichier serait non sensible.

### Minimisation

Déterminer si le besoin opérationnel peut être satisfait par :

- un fingerprint ;
- un code de rejet ;
- un compteur ;
- une preuve technique minimale ;

plutôt que par la conservation du payload complet.

### Chiffrement

Tout futur stockage durable doit imposer :

- chiffrement en transit ;
- chiffrement au repos ;
- gestion de clés séparée du payload ;
- rotation et révocation ;
- interdiction des clés ou secrets dans Git.

### Contrôle d’accès

Le stockage de quarantaine ne doit pas être accessible par défaut aux développeurs, au support général ou aux comptes applicatifs non concernés.

L’accès futur doit être :

- restreint par rôle ;
- justifié ;
- traçable ;
- limité à l’organisation et au fournisseur concernés ;
- révocable.

### Audit

Toute consultation, export, téléchargement, suppression ou replay futur doit produire un audit exploitable, distinct des métriques agrégées.

### Rétention

Aucune durée de conservation n’est codée en dur dans cette tranche.

La durée devra être définie après validation métier, conformité et sécurité, avec :

- date d’expiration calculée ;
- suppression automatique ;
- impossibilité de prolongation silencieuse ;
- preuve de suppression lorsque requise.

### Suppression

La suppression devra couvrir :

- objet principal ;
- copies temporaires ;
- exports ;
- index techniques ;
- caches ;
- sauvegardes selon politique applicable.

### Reprise manuelle

Le replay d’une source rejetée reste désactivé.

Un futur workflow de reprise devra au minimum :

1. ne jamais modifier le fichier original en place ;
2. exiger une nouvelle validation complète ;
3. conserver une relation explicite avec la quarantaine d’origine ;
4. empêcher le double import ;
5. réutiliser les garanties d’idempotence du rapprochement ;
6. auditer l’opérateur, la justification et le résultat.

## Frontière avec l’observabilité actuelle

Le mécanisme existant continue à autoriser uniquement des signaux bornés :

- compteur total de quarantaines ;
- compteur par code fermé ;
- compteur d’échecs ;
- durées d’import ;
- agrégats de rapprochement.

Les métriques ne doivent pas exposer :

- identifiant d’organisation ;
- identifiant fournisseur ;
- référence de fichier ;
- fingerprint ;
- référence transactionnelle ;
- ligne brute ;
- payload rejeté.

## Tests

Les tests ajoutés dans `mansa-platform` vérifient :

- le mode `SIGNALS_ONLY` par défaut ;
- l’absence de stockage brut ;
- l’absence de payload fournisseur durable ;
- l’absence de metadata durable autorisée dans cette phase ;
- l’absence de replay manuel ;
- l’immuabilité du plan retourné ;
- le refus explicite du mode `RAW_SOURCE` ;
- la garde fail-closed dédiée au stockage brut.

## Invariant de sécurité

Tant qu’une tranche future n’implémente pas explicitement et ne teste pas les contrôles décrits ici :

```text
source rejetée
→ métriques bornées
→ erreur de quarantaine
→ aucun stockage brut
→ aucun replay manuel
```

## Prochaine tranche recommandée

Avant de créer un stockage durable, définir un **registre de politiques de quarantaine par fournisseur** capable de représenter les décisions approuvées sans contenir de secrets : classification, mode autorisé, rétention validée, exigences de chiffrement, rôles habilités et statut de replay. Le registre devra rester fail-closed pour tout fournisseur sans politique explicite.
