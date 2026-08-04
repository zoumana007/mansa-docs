# Gouvernance des modèles et décisions d’intelligence artificielle

## 1. Objet

Ce document définit les règles obligatoires pour l’assistant Jini, le scoring de fraude, le triage du support et les recommandations. Il complète les contrats TypeScript de `packages/contracts/src/intelligence.ts`, `intelligence-api.ts` et `ai-governance.ts` du dépôt `mansa-platform`.

## 2. Cas d’usage autorisés

Les cas d’usage techniques reconnus sont :

- `JINI_ASSISTANT` : assistance conversationnelle aux clients et agents ;
- `FRAUD_SCORING` : évaluation du risque d’une transaction, d’un compte, d’un terminal ou d’un commerçant ;
- `SUPPORT_TRIAGE` : classement, résumé et routage des demandes de support ;
- `RECOMMENDATION` : propositions personnalisées sans engagement financier automatique.

Tout nouveau cas d’usage doit être ajouté au catalogue, faire l’objet d’une analyse de risque et être approuvé avant activation.

## 3. Cycle de vie d’un modèle

Un modèle passe uniquement par les états suivants :

1. `DRAFT` : version enregistrée mais non utilisée ;
2. `SHADOW` : calcul en parallèle sans impact utilisateur ;
3. `ACTIVE` : version autorisée en production ;
4. `SUSPENDED` : arrêt immédiat à la suite d’un incident ou d’une dérive ;
5. `RETIRED` : version définitivement retirée.

Le passage en `ACTIVE`, `SUSPENDED` ou `RETIRED` exige une justification, un identifiant idempotent et, pour les environnements sensibles, une demande d’approbation administrative.

## 4. Traçabilité des décisions

Chaque décision automatisée doit conserver au minimum :

- l’identifiant de décision ;
- le cas d’usage ;
- le modèle et sa version ;
- le type et l’identifiant du sujet évalué ;
- le résultat produit ;
- les codes de raisons ;
- le score éventuel ;
- l’identifiant de corrélation ;
- la date de décision ;
- l’indication qu’une revue humaine est requise ou non.

Les journaux ne doivent jamais contenir les secrets des fournisseurs, les documents KYC bruts, les numéros complets de carte ou les codes d’authentification.

## 5. Revue humaine obligatoire

Une intervention humaine est obligatoire lorsque le résultat est :

- `REVIEW` ;
- `STEP_UP` ;
- `BLOCK` ;
- `REFUSE`.

Un modèle ne doit pas, seul, clôturer définitivement un compte, retenir des fonds, refuser une bourse, appliquer une sanction publique ou rejeter un dossier KYC. Il peut recommander une action, déclencher une authentification renforcée ou ouvrir un dossier de revue.

## 6. Données et confidentialité

Chaque version déclare une classification de données : `PUBLIC`, `INTERNAL`, `CONFIDENTIAL` ou `RESTRICTED`.

Règles minimales :

- minimiser les données envoyées au fournisseur ;
- pseudonymiser les identifiants quand le cas d’usage le permet ;
- interdire l’entraînement externe sur les données Mansa sans accord contractuel explicite ;
- définir la durée de conservation par cas d’usage ;
- permettre la suppression ou l’anonymisation selon les obligations applicables ;
- isoler les données par pays et environnement.

## 7. Sécurité de Jini

Jini doit :

- refuser de révéler des secrets, données personnelles d’un tiers ou détails internes de sécurité ;
- ne jamais prétendre qu’une opération financière est réalisée avant confirmation du backend ;
- utiliser uniquement des outils autorisés et des contrats API versionnés ;
- demander une confirmation explicite avant toute action sensible ;
- transmettre au support humain les conversations bloquées, dangereuses ou non résolues ;
- associer chaque réponse à un identifiant de corrélation exploitable par le support.

## 8. Scoring de fraude

Le moteur de risque produit un score, un niveau de risque, des codes de raisons et une action recommandée. Les signaux doivent être versionnés et explicables. Les seuils sont configurables par pays, produit, canal et environnement.

Une version en mode `SHADOW` doit être comparée à la version active avant promotion. Les indicateurs minimaux sont : taux de faux positifs, faux négatifs, rappels en revue manuelle, pertes évitées, latence et disponibilité.

## 9. Surveillance et retour arrière

Chaque modèle actif possède :

- un propriétaire métier ;
- un propriétaire technique ;
- une version précédente réactivable ;
- des seuils d’alerte ;
- un tableau de bord de qualité ;
- une procédure de suspension immédiate ;
- un journal des changements.

La plateforme doit pouvoir désactiver un modèle ou un cas d’usage sans redéploiement complet.

## 10. Critères d’acceptation

- Les états et cas d’usage correspondent exactement à `ai-governance.ts`.
- Toute décision critique déclenche `requiresHumanReview`.
- Une version non `ACTIVE` ne peut influencer une opération réelle.
- Les changements de statut sont idempotents et audités.
- Les réponses de Jini ne peuvent pas créer directement une écriture financière.
- Les traces permettent de retrouver la version utilisée à partir du `correlationId`.
- Aucun secret ou document sensible brut n’apparaît dans les événements de décision.
