# Circuit de double validation

## 1. Finalité

Le circuit de double validation encadre les opérations administratives ou financières qui ne doivent jamais être exécutées par une seule personne. Il complète la politique des accès sensibles et s’applique aux actions marquées comme nécessitant une approbation distincte.

## 2. États

Une demande suit les états suivants :

- `PENDING` : créée et en attente de décision ;
- `APPROVED` : approuvée par une identité autorisée différente de l’initiateur ;
- `REJECTED` : refusée avec un motif obligatoire ;
- `CANCELLED` : annulée par l’initiateur avant décision ;
- `EXPIRED` : arrivée à expiration sans décision.

Un état terminal ne peut plus être modifié.

## 3. Données obligatoires

Chaque demande contient au minimum :

- un identifiant unique ;
- le type d’action et la permission concernée ;
- l’identifiant de l’initiateur ;
- le périmètre pays, organisation, marchand, agence ou ressource ;
- une justification ;
- une date de création et une date d’expiration ;
- l’environnement ciblé ;
- une référence de corrélation vers l’opération métier ;
- le statut courant ;
- l’identité du décideur et son commentaire lorsqu’une décision existe.

Les données sensibles de l’opération ne sont pas copiées inutilement dans la demande. Une référence immuable ou une empreinte doit être utilisée lorsque cela suffit.

## 4. Règles de décision

1. L’initiateur ne peut jamais approuver sa propre demande.
2. Le décideur doit posséder la permission requise et un périmètre compatible.
3. Une demande expirée ne peut plus être approuvée ni rejetée.
4. Un refus exige un motif explicite.
5. Une annulation n’est possible que tant que la demande est en attente.
6. L’exécution métier intervient seulement après une approbation valide.
7. L’approbation ne remplace pas les contrôles métier, de risque, de limite ou de conformité.
8. Toute transition produit un événement d’audit append-only.

## 5. Exécution atomique

L’approbation et l’exécution doivent être liées de façon sûre :

- la demande approuvée est consommée une seule fois ;
- l’opération métier utilise une clé d’idempotence ;
- une reprise après erreur ne crée pas de double exécution ;
- l’état final de l’opération est rattaché à la demande ;
- un échec métier après approbation reste visible et déclenche une reprise contrôlée ou une intervention manuelle.

## 6. Notifications

Le système notifie les acteurs concernés lors de la création, de l’approbation, du refus, de l’annulation, de l’expiration et de l’échec d’exécution. Les notifications ne contiennent aucun secret, document KYC ou donnée financière complète.

## 7. Délais et escalade

Les délais sont configurables par type d’action, environnement et niveau de risque. Une demande proche de l’expiration peut être relancée ou escaladée vers un groupe autorisé, sans contourner la séparation des responsabilités.

## 8. Critères d’acceptation

1. Une auto-approbation est refusée.
2. Une décision sur une demande terminale est refusée.
3. Une demande expirée passe à `EXPIRED` avant toute décision.
4. Un refus sans motif est refusé.
5. Une approbation valide enregistre le décideur et l’horodatage.
6. L’annulation par une autre identité que l’initiateur est refusée.
7. L’exécution d’une demande approuvée est idempotente.
8. Les transitions sont couvertes par des tests unitaires et auditables.
