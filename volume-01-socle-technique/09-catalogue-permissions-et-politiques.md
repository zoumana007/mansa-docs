# Catalogue des permissions et politiques d’autorisation

## Objectif

Ce document normalise les permissions utilisées par les API, applications, tâches asynchrones et interfaces d’administration de Mansa. Il complète les contrats `AuthorizationActor`, `AuthorizationResource`, `AuthorizationContext` et `AuthorizationDecision` du paquet `@mansa/contracts`.

## Convention de nommage

Une permission suit le format :

```text
<domaine>.<ressource>.<action>
```

Exemples :

- `wallet.balance.read`
- `payment.sale.create`
- `merchant.member.invite`
- `terminal.configuration.update`
- `public-fine.payment.collect`
- `admin.feature-flag.update`

Les noms sont en minuscules, séparés par des points et utilisent un tiret uniquement à l’intérieur d’un segment composé. Les permissions génériques comme `admin`, `write` ou `all` sont interdites dans les nouvelles politiques.

## Actions normalisées

- `create` : créer une ressource ou déclencher une commande métier ;
- `read` : consulter une ressource ;
- `list` : consulter une collection ;
- `update` : modifier une ressource ;
- `delete` : supprimer une ressource lorsque la suppression est autorisée ;
- `activate`, `suspend`, `close` : changer explicitement un état métier ;
- `approve`, `reject` : prendre une décision nécessitant une habilitation ;
- `collect` : encaisser une obligation ou un paiement ;
- `export` : produire un fichier ou extraire des données ;
- `impersonate` : ouvrir une session d’assistance contrôlée au nom d’un utilisateur.

Une action métier précise est préférée à `update` lorsqu’elle possède des risques ou obligations spécifiques.

## Portées ABAC

Une permission seule ne suffit pas. La décision doit aussi vérifier les attributs applicables :

- pays ;
- organisation ;
- commerce ;
- point de vente ;
- environnement `DEMO`, `STAGING` ou `PRODUCTION` ;
- niveau d’authentification ;
- niveau KYC ;
- montant et devise ;
- propriété de la ressource ;
- état métier ;
- fenêtre horaire ou règle de risque.

Toute absence d’attribut obligatoire entraîne un refus par défaut.

## Obligations de décision

Une décision autorisée peut imposer une ou plusieurs obligations :

- `REQUIRE_STEP_UP_AUTHENTICATION` ;
- `REQUIRE_FOUR_EYES_APPROVAL` ;
- `REQUIRE_KYC_REVIEW` ;
- `REQUIRE_DEVICE_ATTESTATION` ;
- `MASK_SENSITIVE_FIELDS` ;
- `LIMIT_EXPORT_ROWS` ;
- `WRITE_AUDIT_EVENT` ;
- `NOTIFY_ACCOUNT_OWNER`.

Le service appelant doit appliquer toutes les obligations avant d’exécuter l’action. Une obligation inconnue ou non applicable doit provoquer un refus technique, jamais être ignorée.

## Règles d’évaluation

1. Refuser par défaut.
2. Vérifier l’état de l’identité et de la session.
3. Vérifier le niveau d’authentification requis.
4. Vérifier la permission RBAC.
5. Vérifier les contraintes ABAC de la ressource et du contexte.
6. Vérifier les politiques de risque et de conformité.
7. Calculer les obligations.
8. Journaliser la décision avec le code de motif et les identifiants de politiques évaluées.

Les données sensibles, jetons, secrets et documents KYC ne doivent jamais apparaître dans les traces de décision.

## Codes de motif minimaux

- `ALLOW_POLICY_MATCH` ;
- `DENY_PERMISSION_MISSING` ;
- `DENY_SCOPE_MISMATCH` ;
- `DENY_AUTHENTICATION_LEVEL` ;
- `DENY_ACCOUNT_STATUS` ;
- `DENY_RESOURCE_STATUS` ;
- `DENY_LIMIT_EXCEEDED` ;
- `DENY_RISK_POLICY` ;
- `DENY_ENVIRONMENT_MISMATCH` ;
- `DENY_POLICY_CONFIGURATION_ERROR`.

Les codes sont stables et destinés à l’audit et à l’observabilité. Les messages affichés à l’utilisateur restent génériques lorsque le détail pourrait révéler une règle de sécurité.

## Séparation des environnements

Une habilitation de démonstration ou de recette ne donne aucun accès à la production. Les rôles, affectations, terminaux, clés et politiques sont séparés par environnement. Toute tentative de croisement doit être refusée et auditée.

## Critères d’acceptation

- Une permission inconnue est refusée.
- Une permission valide mais hors portée organisationnelle est refusée.
- Une action sensible avec niveau d’authentification insuffisant retourne une obligation de renforcement ou un refus explicite.
- Une décision contient toujours un `reasonCode` et la liste des politiques évaluées.
- Une obligation non appliquée empêche l’exécution de la commande.
- Les tests automatisés couvrent les catalogues d’acteurs et de niveaux d’authentification.
- Le sous-chemin `@mansa/contracts/authorization` permet d’importer directement les contrats d’autorisation.
