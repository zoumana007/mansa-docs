# Journal d’audit des autorisations

## 1. Objectif

Toute décision d’autorisation sensible de Mansa doit produire un événement d’audit structuré, corrélable et exploitable par les équipes sécurité, conformité, support et contrôle interne. Le journal doit expliquer **qui** a demandé **quelle action**, **dans quelle portée**, **sur quelle ressource**, **dans quel environnement** et **pourquoi** la décision a été autorisée ou refusée.

Le journal d’audit ne remplace ni les journaux techniques ni les écritures du grand livre. Il constitue une preuve de décision d’accès.

## 2. Événements concernés

Un événement est obligatoire pour :

- toute décision portant sur une permission sensible ;
- tout refus d’accès ;
- toute action nécessitant une double validation ;
- toute lecture ou export de données sensibles ;
- toute modification de configuration, frais, limites, partenaire ou politique ;
- toute action administrative sur un compte, un dossier KYC, un paiement, un règlement ou un service public ;
- toute utilisation d’un compte de service en Production.

Les lectures ordinaires à faible risque peuvent être échantillonnées uniquement si la politique réglementaire et la politique interne le permettent. Les refus ne sont jamais échantillonnés.

## 3. Schéma minimal

| Champ | Type | Obligatoire | Description |
|---|---|---:|---|
| `eventId` | chaîne | Oui | Identifiant unique et non réutilisable de l’événement |
| `correlationId` | chaîne | Oui | Identifiant reliant la requête, les logs, les événements et la transaction |
| `occurredAt` | date ISO 8601 UTC | Oui | Date de la décision |
| `actorId` | chaîne | Oui | Identifiant interne de l’acteur |
| `actorType` | enum | Oui | `USER`, `SERVICE_ACCOUNT` ou `SYSTEM` |
| `roles` | tableau | Oui | Rôles évalués au moment de la décision |
| `permission` | chaîne | Oui | Permission demandée |
| `environment` | enum | Oui | `DEMO`, `STAGING` ou `PRODUCTION` |
| `decision` | enum | Oui | `ALLOW` ou `DENY` |
| `reason` | chaîne | Oui | Motif canonique retourné par le moteur d’autorisation |
| `resourceType` | chaîne | Non | Type logique de ressource |
| `resourceId` | chaîne | Non | Identifiant interne ou pseudonymisé de la ressource |
| `resourceOwnerId` | chaîne | Non | Propriétaire utilisé pour la règle d’accès |
| `scope` | objet | Oui | Pays, organisation, commerce, point de vente ou administration concernés |
| `riskLevel` | enum | Non | Niveau de risque évalué |
| `amountMinor` | chaîne entière | Non | Montant en unité mineure, sérialisé sans flottant |
| `currency` | chaîne ISO 4217 | Non | Devise lorsque le montant est présent |
| `requiresDualApproval` | booléen | Oui | Indique si l’action exige une seconde personne |
| `approverActorId` | chaîne | Non | Approbateur distinct lorsque disponible |
| `channel` | chaîne | Non | API, admin web, mobile, TPE, worker ou intégration |
| `metadata` | objet filtré | Non | Références techniques non sensibles autorisées |

## 4. Données interdites

Le journal ne doit jamais contenir :

- mot de passe, code PIN, OTP ou réponse biométrique ;
- clé API, secret, jeton complet, cookie de session ou en-tête d’autorisation ;
- numéro de carte complet, cryptogramme ou données de piste ;
- document KYC brut, photographie ou copie d’identité ;
- contenu intégral d’un message client ;
- coordonnées bancaires complètes non masquées ;
- valeurs libres non filtrées provenant directement du client.

Les identifiants techniques doivent être internes. Les données personnelles doivent être minimisées ou pseudonymisées.

## 5. Intégrité et conservation

1. Les événements sont écrits en ajout uniquement.
2. Une application ne peut pas modifier ou supprimer directement un événement déjà enregistré.
3. Les horodatages serveur font foi.
4. Les événements sont exportés vers un stockage central avec contrôle d’accès séparé.
5. Les accès au journal sont eux-mêmes audités.
6. La durée de conservation est configurable par pays et catégorie, après validation juridique et réglementaire.
7. Les archives doivent être chiffrées, vérifiables et restaurables.
8. Toute rupture de séquence, perte d’événements ou retard d’export déclenche une alerte.

## 6. Corrélation

Le même `correlationId` doit suivre la requête entre l’API Gateway, les services métier, les workers, les adaptateurs partenaires et les notifications. Lorsqu’une opération financière existe, ses identifiants métier sont référencés sans dupliquer les données sensibles.

Une nouvelle tentative idempotente conserve la référence de l’opération initiale mais produit son propre événement de décision lorsque l’autorisation est réévaluée.

## 7. Double validation

Pour une action à double validation, le journal doit distinguer :

- la demande initiale ;
- la décision de mise en attente ;
- l’approbation ou le refus ;
- l’exécution finale ;
- l’expiration ou l’annulation éventuelle.

L’initiateur et l’approbateur doivent être distincts. Toute modification de la demande invalide l’approbation précédente et crée de nouveaux événements.

## 8. Consultation et export

- `AUDITOR` dispose d’un accès en lecture seule limité à sa portée.
- `COMPLIANCE_OFFICER` et `RISK_ANALYST` consultent uniquement les catégories nécessaires à leur mandat.
- `SUPPORT_AGENT` voit une version masquée et ne peut effectuer d’export massif.
- Tout export nécessite `data.export`, un motif, une portée, une durée de validité et, selon le seuil, une double validation.
- Les exports portent un identifiant, un filigrane logique et une date d’expiration.

## 9. Alertes minimales

Une alerte doit être produite pour :

- répétition de refus sur une permission sensible ;
- tentative d’accès inter-pays ou inter-organisation ;
- auto-approbation ;
- accès Production depuis un contexte non autorisé ;
- export massif ou inhabituel ;
- usage anormal d’un compte de service ;
- absence de corrélation ou schéma d’événement invalide ;
- indisponibilité du pipeline d’audit.

## 10. Critères d’acceptation

- Une décision `ALLOW` ou `DENY` peut être convertie en événement structuré déterministe à partir de métadonnées explicites.
- L’événement contient l’acteur, la permission, l’environnement, la portée, la décision et le motif.
- Les montants restent des entiers et sont sérialisés sans perte.
- Aucun champ libre non filtré n’est accepté dans le constructeur canonique.
- Une double validation conserve l’identité de l’approbateur sans permettre l’auto-approbation.
- Les tests couvrent une autorisation, un refus et une opération avec montant.
- Le format partagé est fourni par `mansa-platform/packages/security` et ne doit pas être redéfini différemment dans chaque application.
