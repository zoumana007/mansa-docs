# Volume 5 — Configuration, audit et validations

## 1. Configuration centralisée

L’administration expose une configuration hiérarchique :

```text
valeur globale
  → surcharge pays
    → surcharge partenaire
      → surcharge produit
        → surcharge environnement
```

La valeur effective est calculée par priorité sans écraser la valeur parente. Chaque changement conserve l’ancienne valeur, la nouvelle valeur, la portée, le motif, l’auteur et l’approbateur éventuel.

## 2. Domaines configurables

- Pays, devises, langues, fuseaux horaires et formats.
- Produits et fonctions disponibles.
- Commissions, frais, minimums, maximums et plafonds.
- Niveaux KYC/KYB et limites associées.
- Méthodes de paiement et partenaires autorisés.
- Règles de règlement commerçant.
- Paramètres TPE et versions minimales.
- Modèles de notifications et canaux.
- Délais, seuils de risque et politiques de blocage.
- Textes légaux versionnés et dates d’entrée en vigueur.

## 3. Drapeaux de fonctionnalités

Un drapeau possède :

- une clé stable ;
- un état activé ou désactivé ;
- une portée ;
- une stratégie de déploiement ;
- une période de validité facultative ;
- un propriétaire métier ;
- un plan de retour arrière.

Les stratégies admises sont : activation totale, liste d’utilisateurs, liste d’organisations, pourcentage déterministe, pays, version minimale d’application et environnement.

Toute activation en Production d’une fonction financière nouvelle exige validation secondaire et preuve de tests en Recette.

## 4. Demandes d’approbation

Une demande d’approbation contient :

- type d’action ;
- ressource cible ;
- changement proposé ;
- justification ;
- niveau de risque ;
- initiateur ;
- approbateurs autorisés ;
- date d’expiration ;
- statut et décision.

Statuts : `PENDING`, `APPROVED`, `REJECTED`, `EXPIRED`, `CANCELLED`, `EXECUTED`, `FAILED`.

L’approbation ne modifie pas directement la ressource. Elle autorise une commande d’exécution idempotente qui produit son propre résultat d’audit.

## 5. Journal d’audit

Le journal d’audit enregistre au minimum :

- identifiant unique ;
- horodatage serveur ;
- acteur et type d’acteur ;
- session, adresse réseau et appareil lorsque disponibles ;
- action, ressource et identifiant cible ;
- résultat ;
- motif et référence de dossier ;
- métadonnées filtrées ;
- empreinte de corrélation avec la requête et les événements métier.

Les secrets, mots de passe, codes OTP, numéros complets de carte et documents bruts ne sont jamais copiés dans le journal.

## 6. Consultation et export

- Recherche par acteur, action, ressource, période, pays et résultat.
- Accès aux détails selon autorisation.
- Export signé, limité dans le temps et journalisé.
- Pagination obligatoire et limites renforcées sur les recherches larges.
- Conservation définie par catégorie et obligation réglementaire.

## 7. Gestion des incidents administratifs

Le portail permet :

- blocage immédiat d’une fonction ou intégration ;
- révocation de sessions ;
- suspension ciblée d’un compte, commerçant, terminal ou partenaire ;
- ouverture d’un incident avec gravité, propriétaire et chronologie ;
- association des actions techniques et décisions métier ;
- clôture avec analyse de cause et mesures préventives.

## 8. Critères d’acceptation

- Une configuration invalide est rejetée avant persistance.
- Une surcharge indique clairement sa valeur héritée.
- Chaque modification peut être comparée à la version précédente.
- Les actions à double validation ne peuvent pas être auto-approuvées.
- Une demande expirée ne peut pas être exécutée.
- Tous les exports sensibles sont traçables et expirent automatiquement.
- Les drapeaux peuvent être désactivés immédiatement sans déploiement applicatif.
