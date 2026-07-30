# Volume 6 — Services publics : vision, identité et traçabilité

## 1. Objectif

Le module État permet aux administrations, collectivités, universités et agents habilités d’émettre, encaisser, suivre et rapprocher des obligations publiques dans Mansa. Il couvre notamment les amendes, taxes, frais administratifs, bourses, aides, inscriptions et cartes étudiantes.

Le module ne donne jamais un accès global par défaut. Chaque organisme agit dans un périmètre explicite, avec des rôles, des habilitations, des plafonds et un environnement déterminé.

## 2. Organisations publiques

Chaque organisme public possède :

- un identifiant stable ;
- un type d’organisme ;
- un pays et éventuellement une zone administrative ;
- des comptes de règlement dédiés ;
- des services activés ;
- des règles d’approbation ;
- des agents et superviseurs ;
- des politiques de conservation et d’export.

Un organisme ne peut consulter ou modifier que ses propres dossiers, sauf délégation formelle et auditée.

## 3. Identification des agents

Tout agent utilise une identité personnelle. Les comptes partagés sont interdits.

L’identité agent comprend :

- l’utilisateur Mansa associé ;
- le matricule de l’administration ;
- l’organisme et l’unité d’affectation ;
- les rôles opérationnels ;
- la période de validité ;
- les appareils autorisés ;
- l’état du compte ;
- le niveau d’authentification requis.

Les actions sensibles exigent une authentification renforcée. Une mutation d’affectation, suspension ou révocation doit prendre effet immédiatement.

## 4. Contrôle des droits

Les autorisations combinent :

- RBAC : rôle de l’agent ;
- périmètre organisationnel ;
- périmètre géographique ;
- type de service public ;
- montant et niveau de risque ;
- appareil et environnement ;
- horaires ou conditions opérationnelles ;
- approbation à quatre yeux lorsque nécessaire.

Exemples de permissions :

- `public_obligation.create` ;
- `public_obligation.cancel` ;
- `public_payment.collect` ;
- `public_payment.refund` ;
- `scholarship.approve` ;
- `student_card.issue` ;
- `public_report.export`.

## 5. Dossier public

Toute obligation publique est représentée par un dossier stable contenant :

- une référence lisible ;
- l’organisme émetteur ;
- le service concerné ;
- le bénéficiaire ou redevable ;
- le motif et la base de calcul ;
- le montant en unités mineures et la devise ;
- les dates d’émission, d’échéance et d’expiration ;
- les pièces justificatives référencées ;
- l’état courant ;
- l’historique des transitions.

Une correction ne remplace jamais silencieusement l’historique. Elle crée une nouvelle version ou une opération compensatoire.

## 6. Traçabilité anti-corruption

Chaque action critique produit un événement d’audit comprenant :

- l’identité de l’agent ;
- l’organisme et le rôle actifs ;
- l’appareil et le canal ;
- l’horodatage serveur ;
- le dossier affecté ;
- l’action, son résultat et sa justification ;
- les valeurs avant et après, avec masquage des données sensibles ;
- l’approbation associée, si requise ;
- une corrélation avec la transaction financière.

Les journaux ne sont pas modifiables par les agents métier. Les accès aux journaux sont eux-mêmes audités.

## 7. Paiement sur le terrain

Lorsqu’un agent constate une infraction ou une obligation payable immédiatement :

1. l’agent s’authentifie sur son appareil enrôlé ;
2. il identifie la personne ou saisit les données minimales autorisées ;
3. il sélectionne un motif issu d’un référentiel officiel ;
4. le système calcule ou valide le montant ;
5. un dossier et une référence de paiement sont créés ;
6. la personne choisit un canal de paiement disponible ;
7. Mansa confirme le paiement et émet un reçu vérifiable ;
8. l’opération est rapprochée avec le compte public concerné.

L’agent ne reçoit jamais les fonds sur un portefeuille personnel et ne peut modifier librement le montant réglementaire.

## 8. Mode hors ligne

Le mode hors ligne est limité à la préparation ou à l’enregistrement local signé d’un constat. Le paiement et la confirmation définitive nécessitent une validation serveur, sauf mécanisme certifié et explicitement autorisé.

Les données hors ligne sont chiffrées, expirent rapidement et sont synchronisées avec détection des doublons et conflits.

## 9. Protection des citoyens

- Reçu numérique et référence vérifiable.
- Motif, montant et organisme clairement affichés avant paiement.
- Possibilité de contestation avec numéro de dossier.
- Aucun paiement vers un numéro personnel d’agent.
- Alertes en cas d’annulation, correction ou remboursement.
- Minimisation des données et durées de conservation définies par service.

## 10. Critères de validation

- Aucun agent ne peut agir hors de son organisme ou périmètre.
- Toute obligation et tout paiement disposent d’un historique complet.
- Les montants utilisent le type `Money` commun de la plateforme.
- Les annulations et remboursements sont compensatoires et approuvés selon le risque.
- Les reçus peuvent être vérifiés sans exposer les données personnelles complètes.
