# Catalogue API — Intelligence, Jini et risque transactionnel

## 1. Objet

Ce catalogue décrit les contrats HTTP du premier socle d’intelligence de Mansa. Il couvre l’assistant Jini, l’historique des conversations, l’escalade vers le support et l’évaluation du risque transactionnel.

La source TypeScript correspondante est `packages/contracts/src/intelligence-api.ts` dans `mansa-platform`.

## 2. Principes communs

- Toutes les routes sont versionnées sous `/v1`.
- Les opérations d’écriture exigent une clé d’idempotence.
- Les identifiants techniques sont opaques et ne doivent pas contenir de donnée personnelle.
- Les réponses d’erreur suivent le contrat commun `ApiErrorResponse`.
- Les données envoyées aux services IA doivent être minimisées, filtrées et journalisées selon la politique de confidentialité.
- Une décision de risque ne remplace pas les règles de conformité, les limites transactionnelles ou une décision humaine obligatoire.

## 3. Assistant Jini

### `POST /v1/intelligence/jini/messages`

Crée ou poursuit une conversation avec Jini.

Requête : `AskJiniCommand` avec `idempotencyKey`.

Réponse : conversation mise à jour et message produit par l’assistant.

Contraintes :

- le canal, la langue et le pays sont obligatoires dans le contexte ;
- les réponses peuvent contenir des indicateurs de sécurité et des références de sources ;
- aucune opération financière sensible ne doit être exécutée uniquement sur la base d’un texte libre ;
- lorsqu’une action est demandée, Jini doit produire une intention structurée soumise aux contrôles normaux de l’API.

### `GET /v1/intelligence/jini/conversations/:conversationId`

Retourne les métadonnées d’une conversation autorisée pour l’acteur courant.

### `GET /v1/intelligence/jini/conversations/:conversationId/messages`

Retourne une page de messages. Les paramètres de pagination suivent `PageRequest` et la réponse suit `PageResponse<AiMessage>`.

### `POST /v1/intelligence/jini/conversations/:conversationId/escalation`

Transfère une conversation vers le support humain.

La requête contient le motif et une clé d’idempotence. La réponse contient l’identifiant du ticket de support créé. Une seconde requête avec la même clé ne doit pas créer un nouveau ticket.

## 4. Risque transactionnel

### `POST /v1/intelligence/risk/transactions/evaluate`

Évalue une transaction avant ou pendant son traitement.

Entrées principales : transaction, utilisateur, montant en unité mineure, devise, canal, date, appareil, adresse IP et pays de localisation lorsqu’ils sont disponibles et autorisés.

La réponse `TransactionRiskAssessment` contient :

- un score numérique ;
- un niveau `LOW`, `MEDIUM`, `HIGH` ou `CRITICAL` ;
- une décision `ALLOW`, `REVIEW`, `CHALLENGE` ou `BLOCK` ;
- les signaux explicatifs ;
- la version du modèle ou moteur de règles ;
- la date d’évaluation et une éventuelle expiration.

Une même transaction et une même clé d’idempotence doivent renvoyer la même évaluation logique, sauf révocation explicite par une nouvelle version de décision.

### `GET /v1/intelligence/risk/assessments/:assessmentId`

Retourne une évaluation existante aux seuls rôles autorisés. Les signaux internes sensibles peuvent être masqués selon le profil de l’appelant.

## 5. Autorisations minimales

- Client : créer une interaction Jini et lire ses propres conversations.
- Commerçant : utiliser Jini dans le périmètre de son organisation.
- Support : lire les conversations escaladées et le ticket associé.
- Risque/Fraude : lancer ou consulter une évaluation selon son périmètre.
- Administrateur : consulter les métadonnées et politiques, sans accès automatique au contenu complet des conversations.
- Service de paiement : demander une évaluation technique avec une identité de service dédiée.

## 6. Audit et observabilité

Chaque appel doit conserver : identifiant de corrélation, acteur, finalité, route, décision, version du modèle, latence, résultat et référence d’audit. Les prompts complets, pièces jointes et données personnelles ne doivent pas être copiés dans les logs techniques.

Les métriques minimales sont : taux d’erreur, temps de réponse, taux d’escalade, décisions de risque par niveau, faux positifs confirmés, indisponibilité du moteur et dérive des scores.

## 7. Comportement en cas d’indisponibilité

- Jini peut être désactivé sans bloquer les fonctions financières principales.
- Le moteur de risque suit une politique de repli configurable par produit et montant.
- Une opération à haut risque ne doit jamais être implicitement autorisée parce que le moteur est indisponible.
- Les files de reprise ne doivent pas réexécuter une action financière déjà confirmée.

## 8. Critères d’acceptation

1. Les six routes et méthodes correspondent exactement au contrat TypeScript.
2. Toutes les écritures sont idempotentes.
3. Les conversations sont isolées par utilisateur ou organisation.
4. Une escalade répétée ne crée pas de doublon.
5. Chaque évaluation expose sa version de modèle et ses signaux.
6. Les décisions `CHALLENGE`, `REVIEW` et `BLOCK` déclenchent les parcours configurés.
7. Les journaux ne contiennent ni secret ni contenu personnel non nécessaire.
8. L’indisponibilité du service respecte une politique de repli testée.
