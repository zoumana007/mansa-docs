# Catalogue API — cartes

## 1. Portée

Ce catalogue décrit les routes communes de gestion des cartes Mansa. Les chemins, méthodes et types partagés sont déclarés dans `mansa-platform/packages/contracts/src/card-api.ts` et `card.ts`.

Préfixe : `/v1/cards`.

La carte peut être physique ou virtuelle. Son émission, son réseau, sa personnalisation, son provisionnement dans un portefeuille mobile et son utilisation réelle dépendent d’un émetteur, d’un processeur et de contrats partenaires validés. Mansa ne doit jamais simuler une émission de production sans ces partenaires.

## 2. Création

### `POST /v1/cards`

Crée une demande de carte depuis `CreateCardCommand` et retourne une référence `CardReference`.

Règles minimales :

- exiger une clé d’idempotence ;
- vérifier que le portefeuille appartient au demandeur et peut recevoir une carte ;
- vérifier le niveau KYC, le pays, l’âge, les limites et les politiques du partenaire ;
- sélectionner le type et le réseau uniquement parmi les options autorisées ;
- ne jamais retourner PAN complet, CVV, PIN ou clé de chiffrement ;
- traiter séparément les cartes physiques, virtuelles, temporaires et jetables selon les capacités partenaires ;
- journaliser la demande, la décision et la référence partenaire sans secret ;
- appliquer une authentification renforcée lorsque la politique de risque l’exige.

## 3. Consultation

### `GET /v1/cards`

Liste les cartes autorisées avec pagination et filtres par portefeuille, statut et type.

### `GET /v1/cards/:cardId`

Retourne une référence de carte masquée.

Règles minimales :

- déduire le propriétaire depuis la session pour un client ;
- masquer le numéro de carte et toutes les données sensibles ;
- ne pas révéler l’existence d’une carte hors périmètre ;
- réserver l’accès administratif aux permissions dédiées avec motif auditable ;
- ne jamais mettre en cache non chiffré les données sensibles ;
- distinguer clairement statut Mansa et statut retourné par le processeur.

## 4. Changement de statut

### `PATCH /v1/cards/:cardId/status`

Applique `ChangeCardStatusCommand` pour geler, réactiver, bloquer ou clôturer une carte lorsque la transition est autorisée.

Règles minimales :

- exiger une clé d’idempotence ;
- vérifier la concordance entre l’identifiant du chemin et celui de la commande ;
- appliquer immédiatement le gel local puis synchroniser le processeur ;
- ne pas réactiver automatiquement une carte déclarée volée, expirée ou clôturée ;
- exiger une authentification forte pour les transitions sensibles ;
- notifier le titulaire selon la politique configurée ;
- conserver l’historique complet des transitions et erreurs partenaires.

Une réponse locale ne doit pas être présentée comme définitive tant que le processeur n’a pas confirmé l’action lorsque cette confirmation est requise.

## 5. Contrôles d’usage

### `PATCH /v1/cards/:cardId/controls`

Met à jour les contrôles prévus par `UpdateCardControlsCommand`, par exemple paiements en ligne, sans contact, retraits, paiements internationaux ou catégories marchandes.

Règles minimales :

- exiger une clé d’idempotence ;
- refuser tout contrôle non pris en charge par le partenaire ;
- appliquer le contrôle le plus restrictif entre Mansa, l’émetteur, le réseau et la réglementation ;
- ne pas autoriser un client à contourner un blocage conformité ou fraude ;
- synchroniser l’état avec le processeur et exposer les échecs de synchronisation ;
- journaliser ancien état, nouvel état, acteur et origine.

## 6. Limites

### `PATCH /v1/cards/:cardId/limits`

Met à jour les limites de carte avec `UpdateCardLimitsCommand`.

Règles minimales :

- exiger une clé d’idempotence ;
- représenter tous les montants en unités mineures entières ;
- imposer la même devise que la limite concernée ;
- ne jamais dépasser les plafonds KYC, pays, produit, partenaire ou risque ;
- distinguer plafond demandé, plafond autorisé et plafond effectivement appliqué ;
- imposer une double validation administrative pour les dérogations ;
- tracer toute hausse ou baisse et son motif.

## 7. Sécurité des données de carte

- Le PAN complet, le CVV et le PIN ne doivent pas transiter dans les journaux Mansa.
- Les applications utilisent des jetons du processeur ou de l’émetteur lorsque disponibles.
- L’affichage d’informations sensibles est limité, temporaire, authentifié et audité.
- Les captures d’écran, presse-papiers et notifications doivent être contrôlés sur les écrans sensibles.
- Les secrets cryptographiques restent dans des composants certifiés ou des gestionnaires de clés adaptés.
- Les données de carte ne sont jamais ajoutées aux événements analytics génériques.
- Les exigences PCI DSS applicables doivent être déterminées et validées avant production.

## 8. Résilience et partenaires

- Chaque appel partenaire possède corrélation, délai maximal, reprise contrôlée et circuit breaker.
- Une relance ne doit jamais créer deux cartes ni appliquer deux fois une mutation.
- Les webhooks partenaires sont authentifiés, idempotents et ordonnés par version ou horodatage fiable.
- Les divergences entre Mansa et le processeur alimentent une file de réconciliation.
- Les opérations critiques restent bloquées en cas d’état partenaire inconnu.
- Les environnements Démo, Recette et Production utilisent des identifiants et points d’accès séparés.

## 9. Cohérence technique

La source canonique est constituée de :

- `CARD_API_ROUTES` ;
- `CARD_API_METHODS` ;
- `CardApiContract` ;
- `ListCardsQuery` ;
- les types, statuts et commandes de `card.ts`.

Le sous-chemin `@mansa/contracts/card-api` expose ce catalogue. Les contrôleurs NestJS, applications mobiles, portail administrateur et application TPE doivent importer ces contrats au lieu de redéfinir routes, méthodes ou charges utiles.

## 10. Critères d’acceptation

- Chaque route documentée possède la même méthode et le même chemin dans le catalogue TypeScript.
- Les listes sont paginées et ne retournent que les cartes autorisées.
- Une même clé d’idempotence ne crée pas deux cartes et ne répète pas une mutation.
- Aucune réponse ou journal ne contient PAN complet, CVV, PIN ou secret cryptographique.
- Une carte bloquée, clôturée ou expirée ne peut pas être réactivée par une transition non autorisée.
- Les contrôles et limites ne dépassent jamais les règles plus restrictives du partenaire ou de la conformité.
- Les montants sont exprimés en unités mineures entières.
- Toute divergence partenaire est détectée et réconciliée.
- Toute action sensible enregistre acteur, contexte, résultat et horodatage.
- Les tests couvrent succès, refus, idempotence, concurrence, panne partenaire et ressource hors périmètre.
