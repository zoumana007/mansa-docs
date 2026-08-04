# Catalogue API — terminaux et TPE

## 1. Portée

Ce catalogue décrit les routes du domaine terminaux de paiement. Les chemins, méthodes et types partagés sont déclarés dans `mansa-platform/packages/contracts/src/terminal-api.ts` et `terminal.ts`.

Préfixe : `/v1`.

Le domaine couvre l’enregistrement d’un terminal, son activation, sa configuration, sa supervision et la création d’une vente. Il s’applique aux TPE Android, SoftPOS et terminaux web, avec séparation stricte des environnements Démo, Sandbox et Production.

## 2. Routes

### `POST /v1/terminals`

Enregistre un terminal à partir de `RegisterTerminalCommand` et retourne un `PaymentTerminal`.

Règles minimales :

- exiger une clé d’idempotence au niveau HTTP ;
- vérifier que le commerçant et le point de vente existent et sont accessibles ;
- empêcher la duplication d’un numéro de série dans un même environnement ;
- vérifier que la devise est autorisée pour le commerçant et le pays ;
- initialiser le terminal avec le statut `PENDING_ACTIVATION` ;
- ne jamais enregistrer de secret d’activation en clair.

### `GET /v1/terminals`

Liste les terminaux accessibles selon `ListTerminalsQuery`.

Filtres supportés : `merchantId`, `locationId`, `environment` et `status`, en plus de la pagination commune.

L’autorisation doit toujours limiter les résultats à la portée organisation, point de vente, pays et environnement de l’acteur.

### `GET /v1/terminals/:terminalId`

Retourne un terminal accessible. Une ressource hors portée ne doit pas révéler son existence.

Les réponses ne contiennent jamais de clé privée, secret d’activation, donnée carte sensible ou jeton partenaire.

### `POST /v1/terminals/activate`

Active un terminal avec `ActivateTerminalCommand` et retourne le `PaymentTerminal` mis à jour.

Règles minimales :

- le code d’activation est court, à usage unique et expirant ;
- le numéro de série doit correspondre au terminal enregistré ;
- la clé publique de l’appareil est validée et liée au terminal ;
- l’attestation d’intégrité de l’appareil est vérifiée lorsque le matériel le permet ;
- une activation Production ne peut utiliser aucun identifiant Démo ou Sandbox ;
- l’activation et tout échec sont audités.

### `PUT /v1/terminals/:terminalId/configuration`

Met à jour la configuration avec `UpdateTerminalConfigurationCommand` et retourne le terminal.

La route pilote notamment les moyens de paiement activés, la version minimale de l’application, les pourboires, remboursements et éventuelle capture hors ligne.

Toute modification sensible doit être versionnée, signée côté serveur et récupérée par le terminal. L’activation de la capture hors ligne exige une politique de risque explicite et une limite monétaire.

### `POST /v1/terminals/:terminalId/health`

Accepte un `TerminalHealthReport` et retourne un accusé de réception horodaté.

Le backend contrôle au minimum :

- cohérence entre `terminalId`, certificat appareil et identité de session ;
- version de l’application et version minimale requise ;
- état root/debug ;
- fraîcheur de l’horodatage ;
- version de configuration ;
- fréquence maximale autorisée des rapports.

Un terminal compromis peut être restreint ou suspendu automatiquement selon la politique de risque.

### `POST /v1/terminals/:terminalId/sales`

Crée une vente à partir de `TerminalSaleCommand` et retourne un `Payment`.

Règles minimales :

- vérifier la concordance entre le terminal du chemin et celui de la commande ;
- exiger et dédupliquer `idempotencyKey` ;
- vérifier que l’opérateur est actif et autorisé sur le point de vente ;
- vérifier le statut, l’environnement, la devise et la configuration du terminal ;
- valider le montant et le pourboire dans la même devise ;
- refuser un moyen de paiement non activé ;
- appliquer limites, frais, routage et contrôles antifraude côté serveur ;
- ne jamais considérer une vente comme finalisée avant confirmation du processeur ou du moyen de paiement.

## 3. Statuts et environnements

Les environnements sont définis par `TERMINAL_ENVIRONMENTS` :

- `DEMO`
- `SANDBOX`
- `PRODUCTION`

Les statuts sont définis par `TERMINAL_STATUSES` :

- `PENDING_ACTIVATION`
- `ACTIVE`
- `SUSPENDED`
- `REVOKED`
- `RETIRED`

Les types de terminal sont `ANDROID_POS`, `SOFTPOS` et `WEB_POS`.

Les moyens de paiement autorisables sont définis par `TERMINAL_PAYMENT_METHODS`. Leur disponibilité réelle dépend du pays, du partenaire, de l’environnement, du matériel et de la configuration du commerçant.

## 4. Sécurité et exploitation

- Authentification mutuelle ou preuve d’appareil pour toute route terminal en Production.
- Rotation et révocation des certificats appareil.
- Chiffrement des données en transit et au repos.
- Aucune donnée carte brute dans les journaux, reçus ou événements métier.
- Vérification de l’intégrité applicative et blocage des versions obsolètes.
- Journal d’audit pour enregistrement, activation, configuration, suspension et révocation.
- Corrélation obligatoire entre terminal, opérateur, paiement, tentative de paiement et reçu.
- Limitation de débit distincte pour activation, santé et vente.
- Séparation stricte des clés, identifiants et données entre environnements.
- Mode hors ligne désactivé par défaut et soumis à plafonds, expiration et rapprochement.

## 5. Cohérence technique

La source canonique est constituée de :

- `TERMINAL_API_ROUTES`
- `TERMINAL_API_METHODS`
- `TerminalApiContract`
- `ListTerminalsQuery`
- les commandes et types du fichier `terminal.ts`
- le type de réponse `Payment` du domaine paiements

Le paquet `@mansa/contracts` expose ce catalogue via `@mansa/contracts/terminal-api`. Les contrôleurs NestJS et les applications TPE doivent importer ces contrats au lieu de redéfinir les chemins ou charges utiles.

## 6. Critères d’acceptation

- Chaque route documentée possède la même méthode et le même chemin dans le catalogue TypeScript.
- Un numéro de série ne peut être associé qu’à un terminal autorisé par environnement.
- Une activation est unique, expirante, auditée et liée à une clé publique d’appareil.
- Un terminal ne peut créer une vente que pour son commerçant et son point de vente.
- La création de vente est idempotente.
- Les moyens de paiement et fonctions disponibles proviennent de la configuration serveur.
- Un terminal compromis, révoqué ou obsolète est bloqué conformément à la politique.
- Les applications utilisent les types partagés du paquet de contrats.
