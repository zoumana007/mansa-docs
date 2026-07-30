# Volume 2 — Application Client : parcours et fonctions

## 1. Rôle de l’application

L’application Client est l’interface principale des particuliers. Elle permet de créer et sécuriser un compte, vérifier son identité, gérer ses portefeuilles, payer, transférer, utiliser des cartes, suivre ses opérations et contacter le support.

## 2. Navigation principale

La navigation cible comprend cinq espaces :

1. **Accueil** : solde agrégé, raccourcis, opérations récentes, alertes et recommandations.
2. **Paiements** : QR, transferts, demandes d’argent, Mobile Money, factures et paiements publics.
3. **Cartes** : cartes physiques, virtuelles, temporaires, contrôles, limites et sécurité.
4. **Activité** : historique filtrable, reçus, litiges, exports et détails transactionnels.
5. **Profil** : KYC, sécurité, appareils, préférences, documents, support et paramètres.

Les modules peuvent être masqués ou activés par pays, partenaire, niveau KYC, segment client et drapeau de fonctionnalité.

## 3. Inscription et authentification

Le parcours d’entrée doit prendre en charge :

- inscription par numéro de téléphone ou e-mail selon le pays ;
- validation par OTP avec expiration et limitation des tentatives ;
- création d’un code secret local ;
- biométrie facultative après première authentification forte ;
- détection et gestion des nouveaux appareils ;
- réinitialisation sécurisée avec contrôles renforcés ;
- révocation individuelle ou globale des sessions.

Aucun OTP, mot de passe, code secret ou jeton complet ne doit apparaître dans les journaux.

## 4. Vérification d’identité

Le KYC est progressif. Chaque niveau débloque des plafonds et services définis par configuration.

Le parcours doit permettre :

- saisie des informations personnelles ;
- capture et contrôle des documents acceptés ;
- selfie et contrôle de présence lorsque disponible ;
- reprise d’un brouillon ;
- affichage clair des motifs de rejet ;
- nouvelle soumission sans perdre les informations valides ;
- suivi des statuts : brouillon, soumis, en revue, compléments requis, approuvé ou rejeté.

## 5. Portefeuilles et soldes

Un client peut posséder plusieurs portefeuilles, notamment par devise ou usage. L’écran de solde distingue toujours :

- solde comptable ;
- solde disponible ;
- fonds réservés ;
- devise ;
- date de dernière synchronisation.

Les montants sont affichés selon les conventions locales mais transmis en unités mineures entières.

## 6. Transferts et demandes d’argent

Les transferts internes permettent la recherche du bénéficiaire par identifiant autorisé, la saisie du montant, l’affichage des frais avant confirmation et une authentification adaptée au risque.

Les demandes d’argent permettent :

- création d’un lien ou QR ;
- montant fixe ou libre ;
- date d’expiration ;
- partage par les moyens autorisés ;
- suivi des statuts ;
- annulation tant que la demande n’est pas réglée.

Toute confirmation affiche le bénéficiaire, le montant, la devise, les frais, le total débité et la référence.

## 7. Paiements

L’application doit couvrir progressivement :

- paiement QR statique ou dynamique ;
- paiement à un commerçant ;
- paiement de facture ;
- dépôt et retrait via partenaires ;
- Mobile Money ;
- services publics ;
- paiement récurrent lorsque le partenaire le supporte.

Chaque opération utilise une clé d’idempotence et un état explicite. Une erreur réseau ne doit jamais créer silencieusement un double paiement.

## 8. Cartes

Le client peut consulter et administrer ses cartes selon les droits du programme :

- commander ou créer une carte ;
- activer, geler, dégeler ou résilier ;
- modifier les plafonds ;
- autoriser ou interdire certaines utilisations ;
- consulter les opérations ;
- signaler une perte ou fraude ;
- afficher temporairement les données sensibles après authentification renforcée.

Les données complètes de carte ne sont jamais stockées ni journalisées dans les applications Mansa lorsqu’un fournisseur de tokenisation est utilisé.

## 9. Notifications et centre d’activité

Les notifications utilisent des modèles versionnés et des canaux configurables : application, push, SMS, e-mail ou WhatsApp selon contrat et consentement.

Le centre d’activité doit permettre de :

- distinguer information, action requise et alerte de sécurité ;
- ouvrir directement l’objet concerné ;
- marquer comme lu ;
- conserver une trace cohérente même si un canal externe échoue.

## 10. Support et litiges

Le client peut créer un ticket depuis une transaction ou depuis le centre d’aide. Le ticket contient une catégorie, une priorité calculée ou proposée, des pièces jointes et une conversation historisée.

Les statuts visibles sont traduits en termes simples. Les changements internes, notes privées et données d’autres clients ne sont jamais exposés.

## 11. Accessibilité et qualité

- Interfaces utilisables sur petits écrans et connexions instables.
- États de chargement, vide, erreur et hors ligne définis pour chaque écran.
- Taille de texte adaptable et contrastes suffisants.
- Français prioritaire, avec architecture prête pour d’autres langues.
- Confirmation explicite avant toute action irréversible.
- Aucune dépendance du fonctionnement critique à une animation.

## 12. Critères d’acceptation du premier lot

- Inscription, authentification et gestion de session fonctionnelles en environnement Démo.
- Parcours KYC complet avec statuts simulés.
- Consultation de portefeuille et historique.
- Transfert interne idempotent de bout en bout.
- Paiement QR en mode Démo.
- Gestion de base des cartes simulées.
- Notifications en application et tickets support.
- Tests des parcours critiques et événements d’audit vérifiables.
