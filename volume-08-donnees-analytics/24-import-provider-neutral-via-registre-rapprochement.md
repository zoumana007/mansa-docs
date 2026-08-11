# 24 — Import provider-neutral via registre d’adaptateurs

## Objectif

Cette tranche branche le service d’import de rapprochement sur `ReconciliationProviderRegistry` afin que le pipeline ne dépende plus directement de `TestReconciliationProviderAdapter`.

Le service d’import doit maintenant :

1. recevoir une source normalisée ;
2. lire son `providerId` ;
3. résoudre exactement un adaptateur compatible dans le registre ;
4. préparer le batch avec cet adaptateur ;
5. persister le batch et ses items ;
6. enregistrer l’identifiant technique de l’adaptateur dans les métadonnées ;
7. produire les métriques de succès ou d’échec existantes.

## Nouveau point d’entrée

Le point d’entrée générique est :

```text
importProviderSource(organizationId, source, internalRows)
```

Le service ne connaît plus la classe concrète de l’adaptateur utilisé.

Il dépend uniquement du registre.

## Compatibilité

Le point d’entrée historique :

```text
importTestProviderSource(...)
```

est conservé temporairement afin de ne pas casser les pilotes et tests existants.

Il délègue intégralement à `importProviderSource(...)`.

Aucune logique spécifique au fournisseur de test ne doit rester dans cette méthode de compatibilité.

## Résolution fail-closed

Un `providerId` inconnu doit provoquer une erreur avant toute écriture du batch.

Une ambiguïté de registre doit également provoquer une erreur.

Le service ne doit jamais :

- prendre le premier adaptateur disponible ;
- utiliser l’adaptateur de test comme fallback ;
- inventer un mapping fournisseur ;
- modifier le `providerId` afin de forcer une résolution.

## Métadonnées d’import

Le batch persiste désormais :

```json
{
  "adapter": "<adapterId>"
}
```

La valeur provient du contrat de l’adaptateur résolu, par exemple pour le pilote :

```text
test-normalized-v1
```

Cette métadonnée doit permettre :

- l’audit ;
- le diagnostic ;
- le suivi de versions d’adaptateurs ;
- l’analyse d’un changement de format partenaire.

Elle ne doit contenir aucun secret, credential ou payload brut.

## Monitoring

Le comportement de monitoring existant est conservé :

```text
recordImportStarted
recordImportSucceeded
recordImportFailed
```

Une erreur de résolution d’adaptateur compte comme un échec d’import.

Aucune écriture en repository ne doit avoir lieu si le fournisseur n’est pas résolvable.

## Isolation organisationnelle

Le registre ne choisit jamais `organizationId`.

La portée organisationnelle est fournie séparément au service d’import et transmise au repository.

Un adaptateur n’a pas le droit de changer l’organisation cible à partir des données du fournisseur.

## Sécurité

Cette tranche ne rajoute aucun secret ni connecteur réel.

Les futurs adaptateurs doivent recevoir leurs credentials via leur propre couche d’intégration et configuration, jamais via `ProviderReconciliationSource` et jamais via le registre.

Le code ne doit pas journaliser les lignes brutes du fournisseur lors de la résolution.

## Tests attendus

La tranche couvre :

1. import via le point d’entrée générique ;
2. résolution de l’adaptateur de test à travers le registre ;
3. persistance de `adapterId` dans les métadonnées ;
4. délégation du point d’entrée historique ;
5. rejet d’un fournisseur inconnu ;
6. absence d’appel au repository lors de ce rejet ;
7. enregistrement d’un échec dans le monitoring.

## État après cette tranche

Le pipeline de rapprochement possède désormais une séparation nette :

```text
source fournisseur normalisée
→ registre d’adaptateurs
→ adaptateur compatible unique
→ batch préparé
→ repository
→ métriques / SLO / alerting
```

Le fournisseur de test reste le seul format réellement activé.

## Hors périmètre

Toujours hors périmètre tant qu’aucun format officiel n’est disponible :

- banque réelle ;
- acquéreur réel ;
- opérateur Mobile Money réel ;
- parser ISO 20022 partenaire ;
- ingestion SFTP ;
- webhook partenaire ;
- signature PGP ;
- secrets fournisseur ;
- retry réseau et DLQ propres à une intégration réelle.

## Prochaine tranche recommandée

Après ce branchement, la priorité du rapprochement est de valider le backend partagé réel du port `ReconciliationAlertStateStore` lorsque l’infrastructure cible sera connue, ou de poursuivre l’industrialisation du pipeline d’import avec des frontières explicites d’ingestion, de validation et de quarantaine sans inventer de protocole partenaire.
