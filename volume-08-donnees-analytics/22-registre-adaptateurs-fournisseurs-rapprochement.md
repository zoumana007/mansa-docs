# 22 — Registre d’adaptateurs fournisseurs du rapprochement

## Objectif

Cette tranche prépare l’intégration future de banques, acquéreurs, Mobile Money et autres partenaires sans introduire de format propriétaire ni de secret fournisseur dans le domaine de rapprochement.

Le runtime doit pouvoir sélectionner exactement un adaptateur compatible à partir d’un `providerId`, puis lui transmettre uniquement la source normalisée et les lignes internes nécessaires à la comparaison.

## Principes

Le registre reste provider-neutral :

- aucun nom de banque réel codé en dur ;
- aucun schéma CSV/XML/JSON réel inventé ;
- aucun endpoint externe ;
- aucun token ni secret ;
- aucun fallback implicite vers un adaptateur générique ;
- aucune sélection ambiguë silencieuse.

Un adaptateur réel ne sera ajouté que lorsqu’un contrat partenaire, un échantillon officiel ou une sandbox permettra de connaître son format exact.

## Contrat `ReconciliationProviderAdapter`

Un adaptateur expose :

```text
adapterId
supports(providerId)
prepare(source, internalRows)
```

`adapterId` identifie la version technique de l’adaptateur. Il doit être stable et non vide.

`supports(providerId)` détermine explicitement si l’adaptateur sait traiter un fournisseur donné.

`prepare(...)` retourne le `ReconciliationPreparedBatch` déjà utilisé par le moteur et ne doit pas modifier les invariants financiers du rapprochement.

## Registre

`ReconciliationProviderRegistry` maintient une collection bornée d’adaptateurs enregistrés au démarrage du processus.

Il fournit :

```text
register(adapter)
resolve(providerId)
listAdapterIds()
```

### Enregistrement

Deux adaptateurs ne peuvent pas partager le même `adapterId`.

Un identifiant vide est rejeté.

L’enregistrement est une opération de configuration de runtime, pas une API publique dynamique.

### Résolution

La résolution doit être fail-closed :

- zéro adaptateur compatible -> erreur explicite ;
- exactement un adaptateur compatible -> résolution autorisée ;
- plusieurs adaptateurs compatibles -> erreur explicite.

Le système ne doit jamais choisir « le premier » adaptateur en cas d’ambiguïté.

Cette règle évite qu’un changement d’ordre d’injection ou de configuration fasse traiter un fichier partenaire par le mauvais parseur.

## Adaptateur de test

`TestReconciliationProviderAdapter` reste le seul adaptateur activé dans cette tranche.

Il possède l’identifiant :

```text
test-normalized-v1
```

Il n’accepte que des `providerId` de test explicites, préfixés `TEST` après normalisation de casse et d’espaces.

Il ne doit jamais servir de fallback pour un fournisseur réel.

## Injection NestJS

`ReconciliationModule` construit le registre et y enregistre l’adaptateur de test.

Le registre est exporté par le module afin qu’un futur service d’import partenaire ou worker puisse résoudre l’adaptateur sans dépendre directement d’une classe concrète.

Les futurs adaptateurs réels devront être enregistrés explicitement dans la composition root.

## Sécurité

Le registre ne doit contenir :

- aucun secret fournisseur ;
- aucune configuration de credential ;
- aucune URL d’API ;
- aucun contenu brut de fichier ;
- aucun état transactionnel persistant.

Les credentials futurs devront rester dans une couche d’intégration/configuration dédiée et être injectés hors Git.

Le domaine ne doit jamais journaliser les payloads bruts d’un partenaire au moment de la résolution.

## Isolation tenant

Le registre sélectionne un adaptateur par fournisseur ; il ne décide pas du tenant.

La portée organisationnelle reste imposée par les couches d’identité, de contrôleur, d’import et de repository déjà existantes.

Un adaptateur ne doit pas pouvoir choisir, modifier ou déduire arbitrairement l’organisation cible.

## Versionnement

Lorsqu’un partenaire change de format, créer une nouvelle version technique d’adaptateur plutôt que modifier silencieusement la sémantique d’un `adapterId` déjà déployé.

Exemple générique :

```text
partner-x-statement-v1
partner-x-statement-v2
```

Ces noms sont illustratifs et ne constituent pas des fournisseurs réels configurés.

Une migration de version devra préciser :

- fenêtre d’activation ;
- compatibilité ancienne/nouvelle ;
- stratégie de rollback ;
- validation sur échantillons officiels ;
- métriques d’échec ;
- preuve d’idempotence.

## Tests attendus

La tranche couvre au minimum :

1. résolution de l’unique adaptateur compatible ;
2. rejet d’un fournisseur inconnu ;
3. rejet d’un `providerId` vide ;
4. rejet d’un `adapterId` vide ;
5. rejet d’un doublon d’`adapterId` ;
6. rejet d’une résolution ambiguë ;
7. liste déterministe des identifiants enregistrés ;
8. absence de fallback de l’adaptateur de test vers un fournisseur réel.

## Hors périmètre

Cette tranche ne fournit pas encore :

- adaptateur bancaire réel ;
- parseur CSV partenaire ;
- parseur ISO 20022 ;
- connecteur SFTP ;
- API d’acquéreur ;
- webhook Mobile Money ;
- signature de fichier partenaire ;
- coffre de credentials ;
- rotation de clés ;
- retry réseau ;
- DLQ d’import ;
- mapping de statuts spécifique à un fournisseur.

## Prochaine tranche recommandée

La priorité opérationnelle reste le passage du cooldown d’alerting à un état compatible multi-réplique derrière un port de stockage borné et atomique, sans imposer Redis dans le domaine.

En parallèle, le registre d’adaptateurs est désormais prêt à recevoir un premier adaptateur réel dès qu’un format officiel ou une sandbox partenaire est disponible.
