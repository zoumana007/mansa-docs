# Annuaire Mansa et mini-site commerçant

## 1. Objectif

L’Annuaire Mansa permet aux utilisateurs de découvrir des commerces, services, professionnels et partenaires depuis l’application autonome Annuaire, l’application Client et les interfaces web. Chaque profil publié constitue un mini-site administrable par le commerçant, sous contrôle de modération.

## 2. Périmètre fonctionnel

Le module couvre :

- création d’un profil rattaché à un commerçant et éventuellement à un point de vente ;
- nom public, slug, descriptions courte et complète ;
- catégories, étiquettes et zone géographique ;
- adresse, coordonnées géographiques et horaires d’ouverture ;
- logo, couverture et galerie ;
- contacts publics par téléphone, e-mail, WhatsApp, site web ou messagerie interne ;
- recherche textuelle, géographique et par catégorie ;
- mise en avant temporaire et abonnements sectoriels ;
- modération, suspension et archivage ;
- intégration avec promotions, fidélité, catalogue et paiement lorsque ces modules sont activés.

## 3. États du profil

Les états partagés sont :

- `DRAFT` : brouillon non visible ;
- `PENDING_REVIEW` : soumis à modération ;
- `PUBLISHED` : visible dans les recherches ;
- `SUSPENDED` : masqué temporairement ;
- `ARCHIVED` : fermé définitivement et non réactivable automatiquement.

Transitions autorisées :

- `DRAFT` vers `PENDING_REVIEW` ou `ARCHIVED` ;
- `PENDING_REVIEW` vers `DRAFT`, `PUBLISHED`, `SUSPENDED` ou `ARCHIVED` ;
- `PUBLISHED` vers `SUSPENDED` ou `ARCHIVED` ;
- `SUSPENDED` vers `PENDING_REVIEW`, `PUBLISHED` ou `ARCHIVED` ;
- aucune transition directe depuis `ARCHIVED`.

Toute transition sensible doit enregistrer l’acteur, la date, la justification et la clé d’idempotence.

## 4. Offres d’abonnement

Le socle reconnaît quatre niveaux configurables : `FREE`, `STANDARD`, `PREMIUM` et `ENTERPRISE`.

Les avantages exacts ne sont jamais codés en dur dans l’application. Ils sont définis par pays et période depuis l’administration : nombre de médias, statistiques, priorité dans les résultats, promotions, domaine personnalisé, nombre de points de vente, support et durée de mise en avant.

Un abonnement expiré ne supprime pas le profil. Le système applique la politique de rétrogradation configurée et conserve les données nécessaires à la réactivation.

## 5. Recherche et classement

La recherche peut combiner :

- texte libre ;
- catégories et étiquettes ;
- pays, région administrative et ville ;
- position, rayon et distance ;
- ouverture à une date donnée ;
- profils mis en avant ;
- pagination et tri.

Le classement doit distinguer pertinence organique et mise en avant payante. Toute promotion doit être identifiable par l’interface. La proximité ne doit être calculée que lorsque l’utilisateur a donné son autorisation ou fourni volontairement une zone.

## 6. Données publiques et confidentialité

- Une valeur de contact n’est publique que si le commerçant l’a explicitement publiée.
- Les réponses internes et journaux utilisent une valeur masquée lorsque la donnée complète n’est pas nécessaire.
- Aucun document KYC, identifiant bancaire, numéro de carte, secret ou information privée d’un employé ne peut apparaître sur un profil.
- Les médias sont référencés par identifiant d’actif et servis par un stockage contrôlé.
- Les coordonnées géographiques doivent rester comprises entre -90 et 90 pour la latitude et -180 et 180 pour la longitude.

## 7. API cible

Les contrats partagés se trouvent dans :

- `packages/contracts/src/directory.ts` pour le domaine ;
- `packages/contracts/src/directory-api.ts` pour les routes et méthodes.

Le catalogue API doit permettre au minimum : créer, consulter, modifier, changer le statut, rechercher et lister les profils. Les commandes de création et de changement de statut sont idempotentes.

## 8. Administration et modération

L’administration doit pouvoir :

- examiner les profils en attente ;
- corriger ou refuser des contenus non conformes ;
- suspendre un profil avec motif ;
- gérer catégories, étiquettes et règles par pays ;
- programmer une mise en avant avec date de fin ;
- consulter l’historique des changements ;
- bloquer rapidement une catégorie, un profil ou une zone en cas de risque.

Les actions de publication, suspension, archivage et mise en avant sont auditées. Une double validation peut être imposée selon le niveau de risque.

## 9. Critères d’acceptation

1. Un brouillon n’apparaît jamais dans les résultats publics.
2. Un profil ne peut être publié qu’après validation des champs obligatoires et de la modération.
3. Une recherche géographique refuse les coordonnées invalides.
4. Une mise en avant expirée cesse automatiquement d’influencer le classement.
5. Les contacts privés restent masqués dans les journaux et réponses non autorisées.
6. Deux commandes avec la même clé d’idempotence ne créent pas deux profils ou deux transitions.
7. L’archivage conserve l’audit mais retire le profil des recherches publiques.
8. Les règles d’abonnement sont configurables sans nouvelle version mobile.

## 10. Travaux d’implémentation restants

- persistance PostgreSQL et index géospatial ;
- moteur de recherche et stratégie de classement ;
- stockage et modération des médias ;
- facturation des abonnements et mises en avant ;
- écrans React Native et Next.js ;
- tableaux de bord, statistiques et exports ;
- tests d’intégration, sécurité et performance.
