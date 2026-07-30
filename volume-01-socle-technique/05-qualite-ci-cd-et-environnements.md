# Volume 1 — Qualité, CI/CD et environnements

## 1. Stratégie de branches

La branche `main` reste intégrable et protégée. Toute évolution non triviale passe par une branche courte et une pull request. Les commits suivent une convention lisible (`feat`, `fix`, `docs`, `test`, `build`, `ci`, `chore`).

Aucun déploiement de production ne doit être déclenché depuis un poste développeur. Les livraisons sont produites par une chaîne automatisée et traçable.

## 2. Contrôles obligatoires

Avant fusion, la CI exécute depuis un clone propre :

1. installation déterministe des dépendances ;
2. génération et validation Prisma ;
3. contrôle du formatage ;
4. lint ;
5. vérification TypeScript stricte ;
6. tests unitaires et d’intégration disponibles ;
7. compilation de tous les paquets et applications ;
8. analyse des dépendances et détection de secrets ;
9. validation des contrats OpenAPI lorsque l’API est présente.

Un échec bloque la fusion. Aucun contournement silencieux avec `continue-on-error` n’est accepté sur un contrôle de sécurité ou de compilation.

## 3. Environnements

### Démo

Données synthétiques, partenaires simulés, fonctions clairement marquées et aucun mouvement réel.

### Recette

Environnement proche de la production, comptes de test partenaires, campagnes de validation métier, sécurité et reprise.

### Production

Accès restreint, approbation formelle, secrets gérés, sauvegardes actives, supervision et procédures de retour arrière testées.

Les bases, files, clés, domaines, comptes cloud et identifiants partenaires sont séparés entre environnements.

## 4. Versionnement et publication

- Version sémantique pour les paquets et API.
- Image ou artefact identifié par le SHA du commit.
- Journal des changements généré depuis les pull requests.
- Migrations de base revues, testées sur copie représentative et accompagnées d’un plan de retour.
- Déploiement progressif pour les changements à risque.

## 5. Tests minimums par domaine

- Domaine : invariants, montants, devises, machines à états.
- API : authentification, autorisation, validation, erreurs et idempotence.
- Données : migrations, contraintes, transactions et concurrence.
- Intégrations : signatures, délais, reprises, doublons et indisponibilité.
- Interfaces : parcours critiques, accessibilité et états dégradés.
- Sécurité : permissions, limitation de débit et absence de fuite sensible.

## 6. Critères de livraison

Une version n’est publiable que si :

- tous les contrôles obligatoires réussissent ;
- les migrations sont compatibles avec le déploiement ;
- les tableaux de bord et alertes sont prêts ;
- le plan de retour arrière est documenté ;
- les fonctionnalités sensibles sont protégées par configuration ;
- les responsables métier et technique ont validé la recette.

## 7. Interdictions

- Aucun secret dans GitHub, les images ou les journaux.
- Aucun déploiement direct non audité.
- Aucune modification manuelle de données financières en production.
- Aucune migration destructive sans sauvegarde, validation et procédure dédiée.
