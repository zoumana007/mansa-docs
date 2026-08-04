# Catalogue API — support

## 1. Portée

Ce catalogue décrit les routes communes du service support Mansa : création de ticket, consultation, affectation, mise à jour et messagerie. Les chemins, méthodes et types partagés sont déclarés dans `mansa-platform/packages/contracts/src/support-api.ts` et `support.ts`.

Préfixe : `/v1/support`.

Le module support ne doit pas devenir une voie de contournement des contrôles métier. Un agent peut expliquer, collecter des informations et déclencher une procédure autorisée, mais ne modifie pas directement un solde, une écriture de grand livre, un statut KYC ou une transaction.

## 2. Création d’un ticket

### `POST /v1/support/tickets`

Crée un ticket à partir de `CreateSupportTicketCommand` et retourne un `SupportTicket`.

Règles minimales :

- exiger une clé d’idempotence ;
- vérifier que le demandeur peut ouvrir un ticket pour la ressource liée ;
- limiter la longueur du sujet et de la description ;
- interdire les secrets, codes PIN, mots de passe, PAN complets et données KYC inutiles ;
- contrôler les pièces jointes avant leur association ;
- attribuer une référence publique non prédictible ;
- calculer la priorité initiale selon la catégorie, le risque et l’impact ;
- corréler le ticket à la transaction ou ressource concernée sans dupliquer les données sensibles ;
- journaliser la création et l’origine de la demande.

Les catégories partagées couvrent notamment compte, KYC, transfert, paiement, carte, commerçant, terminal, service public, sécurité et autres demandes.

## 3. Consultation

### `GET /v1/support/tickets`

Liste les tickets accessibles avec pagination et filtres :

- demandeur ;
- agent assigné ;
- catégorie ;
- priorité ;
- statut ;
- période de création.

Un client ne voit que ses propres tickets. Un agent ne voit que les périmètres autorisés par son rôle, son pays, son organisation et son équipe. Les champs internes restent exclus des réponses destinées au client.

### `GET /v1/support/tickets/:ticketId`

Retourne un ticket autorisé. Une ressource hors périmètre ne doit pas révéler son existence.

Les messages internes, annotations de fraude, informations d’investigation et données masquées ne sont exposés qu’aux rôles explicitement autorisés.

## 4. Mise à jour

### `PATCH /v1/support/tickets/:ticketId`

Met à jour les propriétés autorisées avec `UpdateSupportTicketCommand`, complété par l’acteur, le motif et une clé d’idempotence.

Règles minimales :

- vérifier la concordance entre l’identifiant du chemin et celui de la commande ;
- contrôler les transitions de statut ;
- exiger un motif pour toute affectation, escalade, résolution, fermeture ou réouverture ;
- interdire la fermeture d’un ticket bloqué par une action obligatoire ;
- empêcher un agent de s’auto-attribuer un ticket sensible lorsque la séparation des tâches l’interdit ;
- tracer l’ancien état, le nouvel état, l’acteur, le motif et l’horodatage ;
- notifier le client uniquement lorsque le changement est visible et utile.

Cycle de vie partagé : `OPEN`, `IN_PROGRESS`, `WAITING_CUSTOMER`, `WAITING_PARTNER`, `RESOLVED`, `CLOSED`.

Une résolution indique qu’une réponse ou solution a été fournie. La fermeture indique la fin administrative du dossier. Une réouverture doit créer une nouvelle trace d’audit.

## 5. Messages

### `POST /v1/support/tickets/:ticketId/messages`

Ajoute un message avec `AddSupportMessageCommand` et retourne un `SupportMessage`.

Règles minimales :

- exiger une clé d’idempotence ;
- vérifier que l’auteur peut accéder au ticket ;
- distinguer strictement les messages visibles du client et les notes internes ;
- interdire les notes internes pour les clients ;
- analyser les pièces jointes avant mise à disposition ;
- appliquer masquage, antivirus, quotas, taille et types autorisés ;
- conserver l’auteur réel et son type `CUSTOMER`, `AGENT` ou `SYSTEM` ;
- horodater côté serveur ;
- empêcher la modification silencieuse d’un message déjà envoyé.

Une correction doit produire une nouvelle version ou une trace d’édition auditable.

## 6. Priorités et délais

Les priorités partagées sont `LOW`, `NORMAL`, `HIGH` et `URGENT`.

La priorité ne doit pas dépendre uniquement du choix du client. Le moteur de règles peut l’élever selon :

- suspicion de fraude ou compromission ;
- argent bloqué ;
- indisponibilité générale ;
- impact commerçant ou service public ;
- vulnérabilité réglementaire ;
- répétition d’un incident ;
- seuil financier configurable.

Chaque priorité est associée à des objectifs de prise en charge et de résolution configurables par pays, produit, partenaire et plage horaire.

## 7. Sécurité et confidentialité

- Les agents voient uniquement les données nécessaires à leur mission.
- Les coordonnées, identifiants de paiement et documents sont masqués par défaut.
- Les téléchargements utilisent des liens signés, courts et contrôlés.
- Les pièces jointes sont stockées hors du message, chiffrées et analysées.
- Toute consultation sensible est auditée.
- Les exports massifs sont interdits sans permission dédiée et justification.
- Les actions financières restent dans leurs modules d’origine avec leurs contrôles d’autorisation.
- Les notes internes ne doivent jamais être envoyées au client par erreur.
- Les journaux techniques ne doivent pas contenir le corps complet des messages ni des pièces jointes.

## 8. Résilience et exploitation

- Les notifications de nouveau message sont asynchrones et ne bloquent pas l’écriture du ticket.
- Les événements support sont corrélés avec les opérations concernées.
- Les erreurs de notification n’annulent pas la création du message.
- Les files de traitement utilisent retry borné et quarantaine.
- Les métriques couvrent volume, ancienneté, délai de première réponse, résolution, réouverture, escalade et satisfaction.
- Les alertes détectent les tickets urgents non pris en charge, les files anormales et les délais dépassés.
- Les règles d’archivage et de conservation sont configurables selon la catégorie et les obligations applicables.

## 9. Cohérence technique

La source canonique est constituée de :

- `SUPPORT_API_ROUTES` ;
- `SUPPORT_API_METHODS` ;
- `SupportApiContract` ;
- `ListSupportTicketsQuery` ;
- `AddSupportMessageCommand` ;
- les types du fichier `support.ts`.

Le paquet `@mansa/contracts` expose ce catalogue via `@mansa/contracts/support-api`. Les contrôleurs NestJS, applications mobiles, portail administrateur et workers doivent importer ces contrats au lieu de redéfinir les chemins ou charges utiles.

## 10. Critères d’acceptation

- Chaque route documentée possède la même méthode et le même chemin dans le catalogue TypeScript.
- Une même clé d’idempotence ne crée pas plusieurs tickets ou messages.
- Un client ne peut consulter ni modifier le ticket d’un autre client.
- Une note interne n’est jamais exposée au client.
- Les transitions de statut sont contrôlées et auditées.
- Les pièces jointes sont analysées, limitées et servies par accès temporaire.
- Les agents ne peuvent pas contourner les autorisations des modules financiers.
- Les priorités et délais sont configurables et mesurables.
- Les actions sensibles enregistrent acteur, motif, portée et horodatage.
- Les métriques permettent de détecter les files en retard et les incidents urgents.
