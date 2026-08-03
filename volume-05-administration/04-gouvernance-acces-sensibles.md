# Gouvernance des accès sensibles

## 1. Finalité

Cette politique complète la matrice des rôles et permissions. Elle fixe les contrôles obligatoires pour les accès administratifs, les opérations financières exceptionnelles, les données personnelles et les changements de configuration en environnement de production.

## 2. Accès privilégiés

Un accès privilégié est tout accès permettant au minimum l’une des actions suivantes :

- modifier une règle de frais, de limite ou de risque ;
- suspendre ou débloquer un utilisateur ;
- consulter un document KYC sensible ;
- initier ou approuver un ajustement du grand livre ;
- modifier un partenaire ou un moyen de règlement ;
- activer une fonction en production ;
- exporter des données personnelles ou financières ;
- administrer un service public ou ses agents.

Chaque accès privilégié doit être nominatif, limité dans le temps lorsque possible, associé à un périmètre explicite et protégé par une authentification multifacteur.

## 3. Cycle de vie d’un accès

1. Une demande précise le rôle, le périmètre, la justification et la durée.
2. Le responsable métier valide le besoin.
3. La sécurité ou la conformité valide les droits sensibles.
4. L’accès est créé avec le minimum de permissions.
5. Une notification est envoyée au titulaire et au responsable.
6. L’accès est revu périodiquement et à chaque changement de poste.
7. L’accès est retiré immédiatement lors d’un départ, d’une suspension ou d’un incident.

Les comptes partagés sont interdits, sauf compte technique documenté ne permettant aucune connexion interactive.

## 4. Authentification renforcée

Les administrateurs doivent utiliser :

- une authentification multifacteur résistante au hameçonnage lorsqu’elle est disponible ;
- une session courte avec renouvellement contrôlé ;
- une réauthentification avant une action critique ;
- un appareil enregistré ou une politique d’accès conditionnel ;
- des moyens de récupération distincts et audités.

Les jetons d’administration ne doivent jamais être accessibles depuis les journaux, les URLs, les outils analytiques ou les applications clientes.

## 5. Double validation

La double validation est obligatoire pour :

- les ajustements comptables ;
- les remboursements exceptionnels au-dessus d’un seuil configurable ;
- l’activation d’un partenaire financier en production ;
- les modifications sensibles des frais et limites ;
- les exports massifs de données ;
- la remise en service d’un compte gelé pour fraude ou conformité ;
- l’annulation administrative d’un dossier public déjà encaissé.

L’initiateur et l’approbateur doivent être deux identités distinctes. L’approbateur doit disposer de la permission appropriée et d’un périmètre compatible.

## 6. Accès d’urgence

Un mécanisme d’accès d’urgence peut être prévu pour restaurer un service critique. Il doit :

- être désactivé par défaut ;
- nécessiter une justification et une validation renforcée ;
- avoir une durée très courte ;
- déclencher une alerte immédiate ;
- enregistrer toutes les actions ;
- imposer une revue après incident ;
- provoquer la rotation des moyens d’authentification utilisés.

L’accès d’urgence ne doit pas permettre de supprimer ou d’altérer les journaux.

## 7. Journalisation

Pour toute action sensible, le journal contient au minimum :

- identifiant de l’acteur et rôles effectifs ;
- permission demandée ;
- ressource et périmètre ;
- valeur avant et valeur après lorsqu’applicable ;
- justification et référence de ticket ;
- approbateur éventuel ;
- adresse réseau, appareil ou identité de service ;
- environnement ;
- date, identifiant de corrélation et résultat.

Les journaux sont append-only, exportés vers un stockage séparé et soumis à une politique de rétention validée juridiquement.

## 8. Revues périodiques

- Les comptes privilégiés sont revus au minimum chaque trimestre.
- Les comptes inactifs sont suspendus automatiquement selon un délai configurable.
- Les permissions rarement utilisées sont signalées pour retrait.
- Les comptes de service sont revus avec leur propriétaire, leur intégration et leur date de rotation.
- Les écarts entre la documentation et `@mansa/security` bloquent la mise en production.

## 9. Critères d’acceptation

1. Aucun compte privilégié ne peut être anonyme ou partagé.
2. Une action critique exige une authentification récente.
3. Une auto-approbation est systématiquement refusée.
4. Un accès hors périmètre est refusé même si le rôle possède la permission.
5. Toute modification sensible laisse une trace exploitable et non modifiable.
6. Une révocation prend effet sur les nouvelles requêtes et invalide les sessions concernées.
7. Un accès d’urgence déclenche une alerte et une revue obligatoire.
8. Les tests vérifient les refus aussi bien que les autorisations.
