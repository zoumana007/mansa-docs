# Contrôles de vélocité

## Objectif

Les contrôles de vélocité limitent la répétition anormale d’actions sensibles sur une période courte ou cumulée. Ils complètent les plafonds financiers, le moteur de risque, la confiance de l’appareil et la confiance du bénéficiaire.

Le contrat technique de référence se trouve dans `mansa-platform/packages/security/src/velocity.ts`.

## Opérations couvertes

Le socle prend en charge les familles suivantes :

- `LOGIN` : tentatives de connexion ;
- `OTP_REQUEST` : demandes de code à usage unique ;
- `TRANSFER` : transferts entre comptes ou vers un bénéficiaire ;
- `PAYMENT` : paiements marchands ;
- `CASH_OUT` : retraits et décaissements ;
- `BENEFICIARY_CREATE` : création de bénéficiaires.

De nouvelles opérations doivent être ajoutées au contrat partagé avant d’être utilisées par une application.

## Fenêtres de contrôle

Chaque règle s’applique sur une fenêtre `MINUTE`, `HOUR` ou `DAY`. Une même opération peut cumuler plusieurs règles, par exemple trois transferts par minute et un montant total quotidien configurable.

## Règles métier

1. Une règle possède un identifiant stable, une opération, une fenêtre, un état actif et au moins un seuil utile.
2. Les nombres d’opérations sont des entiers positifs.
3. Les montants sont exprimés en unités mineures avec `bigint` dans le domaine TypeScript.
4. Le seuil est considéré atteint lorsque le compteur est supérieur ou égal à la limite configurée.
5. Une règle désactivée ne bloque aucune opération.
6. L’évaluation doit retourner l’identifiant exact de la première règle bloquante.
7. Les compteurs sont isolés au minimum par acteur, opération et fenêtre. Selon le risque, ils peuvent aussi être isolés par appareil, bénéficiaire, commerçant, adresse réseau, agence ou pays.
8. Les mises à jour de compteurs doivent être atomiques pour éviter qu’un traitement concurrent contourne les seuils.
9. Les décisions bloquantes et les modifications de règles sont journalisées.
10. Les règles de Production ne sont modifiables que par un rôle autorisé et peuvent nécessiter une double validation.

## Ordre d’intégration

Pour une opération financière, le service applicatif :

1. authentifie et autorise l’acteur ;
2. charge les compteurs et règles applicables ;
3. appelle `evaluateVelocity` ;
4. interrompt l’opération en cas de refus ;
5. poursuit les contrôles de limites, bénéficiaire, appareil et risque ;
6. réserve ou incrémente atomiquement les compteurs au moment défini par la transaction métier ;
7. produit un événement d’audit corrélé à la transaction.

La fonction pure du package sécurité ne persiste rien. La persistance, la réservation atomique et l’expiration des fenêtres appartiennent aux services et adaptateurs d’infrastructure.

## Administration

L’administration doit permettre de configurer les règles par environnement, pays, produit, segment KYC, type d’acteur et canal. Toute modification affiche l’ancienne valeur, la nouvelle valeur, l’auteur, l’approbateur éventuel, la justification et la date d’effet.

Une baisse importante d’un seuil peut bloquer des opérations légitimes. Elle doit donc pouvoir être simulée en Démo ou Recette avant activation en Production.

## Critères d’acceptation

- Une opération sous les seuils retourne `VELOCITY_ALLOWED`.
- Le nombre maximal atteint retourne `COUNT_LIMIT_EXCEEDED` avec l’identifiant de règle.
- Le montant maximal atteint retourne `AMOUNT_LIMIT_EXCEEDED` avec l’identifiant de règle.
- Une règle désactivée est ignorée.
- Une règle sans compteur correspondant est ignorée.
- Les tests unitaires couvrent les décisions d’autorisation et de blocage.
- Aucun montant flottant ni secret n’est introduit.
