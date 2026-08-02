# Sessions, jetons et révocation

## Objectif

Le système d’authentification doit limiter l’impact d’un jeton compromis, permettre la révocation immédiate et fournir une politique commune aux applications Client, Commerçant, TPE, Admin Lite et portails web.

## Modèle de session

Une session représente une authentification active sur un appareil ou un navigateur. Elle contient au minimum :

- un identifiant opaque de session ;
- l’identifiant de l’acteur ;
- le type de canal ;
- l’environnement ;
- la date de création, la dernière activité et l’expiration absolue ;
- l’état de révocation ;
- une empreinte non réversible de l’appareil lorsque le canal le permet ;
- le niveau d’assurance atteint lors de l’authentification.

Les données biométriques brutes et les codes PIN ne sont jamais stockés par Mansa. La biométrie reste gérée par le système sécurisé de l’appareil.

## Jetons

- Les jetons d’accès sont courts et ne servent qu’à appeler les API.
- Les jetons de rafraîchissement sont rotatifs et liés à une session.
- Chaque rotation invalide le jeton de rafraîchissement précédent.
- La réutilisation d’un ancien jeton déclenche la révocation de toute la famille de jetons.
- Aucun jeton n’est écrit dans les journaux applicatifs.
- Les secrets de signature et clés privées proviennent du gestionnaire de secrets.

## Niveaux d’assurance

- `BASIC` : authentification standard réussie.
- `STRONG` : second facteur ou authentification forte récente.
- `HARDWARE_BOUND` : authentification liée à un terminal ou matériel approuvé.

Les opérations à risque élevé peuvent exiger une élévation temporaire du niveau d’assurance.

## Motifs de révocation

- déconnexion volontaire ;
- expiration ;
- changement ou réinitialisation du secret d’authentification ;
- appareil déclaré perdu ;
- compte suspendu ou fermé ;
- détection de réutilisation d’un jeton ;
- décision de sécurité ou de conformité ;
- rotation administrative globale.

## Règles par canal

### Mobile Client et Commerçant

Le jeton de rafraîchissement est conservé dans le stockage sécurisé du système. Une validation locale par PIN ou biométrie peut protéger l’ouverture de l’application, sans remplacer l’autorisation serveur.

### TPE Android

La session est liée au terminal enregistré. Les opérations sensibles exigent un terminal actif, non bloqué et affecté au bon commerce. Le mode hors ligne n’autorise que les fonctions explicitement prévues par la politique TPE.

### Administration

Les sessions administratives ont une durée plus courte, nécessitent une authentification forte et sont révoquées lors d’un changement de rôle ou de portée. Les actions critiques restent soumises à double validation lorsque configuré.

## Contrat technique partagé

Le paquet `packages/security` expose les types de session, les états, les motifs de révocation et une fonction pure d’évaluation. Cette fonction refuse notamment :

- une session révoquée ;
- une session arrivée à expiration absolue ;
- une session inactive au-delà de la durée autorisée ;
- un environnement différent ;
- un niveau d’assurance insuffisant ;
- un appareil différent lorsqu’une liaison matérielle est exigée.

## Audit

Toute création, rotation, élévation, révocation et détection de réutilisation génère un événement d’audit corrélé. Les journaux ne contiennent ni jeton, ni secret, ni donnée biométrique.

## Critères d’acceptation

1. Une session révoquée est refusée immédiatement.
2. Les expirations absolue et d’inactivité sont évaluées côté serveur.
3. Un jeton destiné à Démo ne peut être accepté en Recette ou Production.
4. Une opération exigeant `STRONG` refuse une session `BASIC`.
5. Une session liée à un appareil refuse une empreinte différente.
6. La rotation et la révocation sont idempotentes.
7. Les tests couvrent les décisions d’acceptation et chaque motif de refus.
