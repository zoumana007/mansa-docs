# Volume 2 — Cartes physiques et virtuelles

## 1. Objectif

Le module Cartes permet au client de consulter, créer et administrer ses cartes physiques et virtuelles depuis l’application Mansa. L’émission réelle dépend toujours d’un partenaire agréé et d’un processeur de cartes configuré.

## 2. Types de cartes

- Carte physique nominative.
- Carte virtuelle permanente.
- Carte virtuelle temporaire avec expiration courte.
- Carte virtuelle jetable renouvelée après une autorisation réussie.

Chaque produit de carte est activable par pays, partenaire, niveau KYC et segment client.

## 3. États

Une carte peut être :

- `PENDING` : création demandée au processeur ;
- `ACTIVE` : utilisable selon ses limites ;
- `FROZEN` : temporairement bloquée par le client ou un administrateur ;
- `BLOCKED` : bloquée à la suite d’un contrôle risque, conformité ou sécurité ;
- `EXPIRED` : date d’expiration dépassée ;
- `CANCELLED` : clôturée définitivement.

Une carte annulée ou expirée ne peut jamais redevenir active. Un déblocage ne doit être possible que pour une carte gelée et après contrôle des droits de l’acteur.

## 4. Données affichées

L’application affiche uniquement les données nécessaires :

- nom commercial du produit ;
- réseau ;
- quatre derniers chiffres ;
- mois et année d’expiration ;
- état ;
- portefeuille débité ;
- limites et consommation ;
- fonctions activées : paiement en magasin, en ligne, sans contact et retrait.

Le PAN complet, le cryptogramme et le code PIN ne sont jamais stockés dans les contrats partagés ni journalisés. Leur révélation éventuelle passe par un composant sécurisé fourni par le processeur, avec authentification forte et durée limitée.

## 5. Commandes client

Le client peut, selon le produit et ses autorisations :

- demander une nouvelle carte ;
- geler ou dégeler une carte ;
- modifier les usages autorisés ;
- définir des plafonds inférieurs aux plafonds réglementaires ;
- signaler une perte, un vol ou une fraude ;
- demander le remplacement ou l’annulation.

Toutes les commandes sensibles utilisent une clé d’idempotence et produisent un événement d’audit.

## 6. Limites

Les limites sont exprimées en unités monétaires mineures et séparées par usage :

- paiement par opération ;
- paiement journalier ;
- retrait par opération ;
- retrait journalier ;
- paiement en ligne journalier.

La limite effective est la plus restrictive entre la configuration du produit, la réglementation, le niveau KYC, le contrôle risque, le portefeuille et le choix du client.

## 7. Sécurité et fraude

- Authentification forte avant révélation de données sensibles ou changement critique.
- Blocage immédiat disponible depuis l’application et l’administration.
- Détection de vélocité, pays inhabituel, terminal inhabituel et montant atypique.
- Notifications en temps réel pour autorisations, refus, gel et dégel.
- Aucune donnée PCI sensible dans les journaux, analytics ou notifications.
- Les jetons du processeur sont conservés dans un stockage chiffré côté serveur, jamais dans les applications clientes.

## 8. Cohérence avec le code

Les contrats partagés sont définis dans `packages/contracts/src/card.ts` et exportés depuis `packages/contracts/src/index.ts`. Ils ne contiennent que les références non sensibles nécessaires aux interfaces et aux modules métier.

## 9. Critères d’acceptation

- Les quatre types de cartes sont distingués par le contrat.
- Les transitions finales sont identifiables.
- Les limites utilisent le type `Money`.
- Les commandes utilisent une clé d’idempotence.
- Aucun PAN, PIN ou cryptogramme n’apparaît dans les objets partagés.
- Une interface peut afficher l’état et les contrôles sans dépendre du processeur choisi.
