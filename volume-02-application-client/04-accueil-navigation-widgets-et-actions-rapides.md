# Volume 02 — Application Client

# Chapitre 04 — Accueil, navigation, widgets et actions rapides

## 1. Objectif

L'écran d'accueil est le cœur de Mansa.

En moins de deux secondes, l'utilisateur doit pouvoir :

- consulter son patrimoine ;
- voir son solde ;
- envoyer de l'argent ;
- recevoir de l'argent ;
- retrouver une conversation ;
- consulter ses cartes ;
- lancer un paiement NFC ;
- scanner un QR Code ;
- voir ses dépenses ;
- accéder aux services publics ;
- retrouver Jini ;
- consulter les alertes importantes.

L'écran doit rester extrêmement fluide même sur un téléphone d'entrée de gamme.

## 2. Philosophie UX

L'accueil doit être :

- plus rapide que Revolut ;
- plus simple que PayPal ;
- plus personnalisable qu'Apple Wallet ;
- plus intelligent grâce à Jini ;
- entièrement configurable par l'utilisateur.

Aucun écran inutile.

Maximum deux clics pour une action fréquente.

## 3. Structure générale

Ordre recommandé :

1. Header ;
2. Solde ;
3. Cartes ;
4. Actions rapides ;
5. Activité récente ;
6. Widgets personnalisés ;
7. Jini ;
8. Promotions ;
9. Services publics ;
10. Actualités ;
11. Barre de navigation.

## 4. Header

Le Header contient :

- photo utilisateur ;
- prénom ;
- identifiant Mansa ;
- cloche notifications ;
- recherche universelle ;
- bouton QR ;
- paramètres rapides.

Exemple :

```text
Bonjour Zoumana 👋

@mansa-zou

🔔      🔍      QR
```

## 5. Solde

Le bloc principal affiche :

- solde global ;
- solde disponible ;
- solde bloqué ;
- devise principale ;
- dernière mise à jour.

Fonctions :

- masquer ;
- afficher ;
- copier ;
- convertir ;
- changer de devise ;
- détail.

Animation : le montant s'anime lors des mises à jour.

## 6. Comptes

Tous les comptes apparaissent.

Exemples :

- Compte Principal ;
- Compte Épargne ;
- Compte USD ;
- Compte EUR ;
- Compte Professionnel ;
- Compte Investissement.

L'utilisateur peut :

- changer l'ordre ;
- masquer ;
- épingler ;
- renommer.

## 7. Cartes

Le carrousel présente notamment :

- Carte Premium ;
- Carte Virtuelle ;
- Carte Temporaire ;
- Carte Enfant ;
- Carte Entreprise.

Actions :

- verrouiller ;
- déverrouiller ;
- voir PIN ;
- plafond ;
- historique ;
- NFC ;
- ajouter Wallet.

Animation : les cartes tournent légèrement lors du défilement.

## 8. Actions rapides

Première ligne :

- Envoyer ;
- Recevoir ;
- Scanner ;
- NFC.

Deuxième ligne :

- Demander ;
- Recharge ;
- Retrait ;
- Paiement.

Troisième ligne :

- Carte virtuelle ;
- Ajouter bénéficiaire ;
- Partager QR ;
- Messagerie.

Toutes les actions sont configurables.

## 9. Recherche universelle

Une seule barre recherche :

- utilisateur ;
- commerçant ;
- entreprise ;
- administration ;
- transaction ;
- facture ;
- carte ;
- conversation ;
- document ;
- investissement ;
- service public ;
- établissement scolaire.

Résultats instantanés.

## 10. Activité récente

Timeline moderne.

Regroupement automatique :

- Aujourd'hui ;
- Hier ;
- Cette semaine ;
- Ce mois.

Chaque opération contient :

- icône ;
- montant ;
- statut ;
- bénéficiaire ;
- catégorie ;
- heure ;
- reçu.

## 11. Widgets

Widgets disponibles :

- Dépenses ;
- Revenus ;
- Budget ;
- Coffres ;
- Objectifs ;
- Investissements ;
- Fidélité ;
- Cashback ;
- Promotions ;
- Cartes ;
- Favoris ;
- Mansa Connect ;
- Jini ;
- Services Publics ;
- Annuaire ;
- Taux de change ;
- Crypto si activée ;
- Marchés ;
- Factures ;
- Documents.

## 12. Personnalisation

Chaque widget peut être :

- déplacé ;
- supprimé ;
- ajouté ;
- réduit ;
- agrandi ;
- verrouillé.

La configuration est synchronisée sur tous les appareils.

## 13. Mansa Connect

Depuis l'accueil :

- conversations récentes ;
- demandes d'argent ;
- paiements récents ;
- contacts favoris ;
- messages non lus.

Un clic ouvre directement la conversation.

## 14. Assistant IA Jini

Carte dédiée.

Exemples de messages :

- « Tu as dépensé 18 % de moins ce mois. »
- « Attention, ton abonnement Netflix augmente. »
- « Tu peux économiser 12 000 FCFA. »
- « Une facture arrive demain. »

## 15. Alertes

Alertes prioritaires :

- carte expirée ;
- nouvelle connexion ;
- KYC ;
- maintenance ;
- promotion ;
- investissement ;
- document ;
- support ;
- État.

## 16. Navigation principale

Cinq onglets :

- Accueil ;
- Paiements ;
- Cartes ;
- Messages ;
- Profil.

Configuration possible par l'administration.

## 17. États

- premier lancement ;
- aucun compte ;
- aucune transaction ;
- maintenance ;
- connexion lente ;
- mode hors ligne ;
- erreur serveur ;
- synchronisation.

## 18. Hors ligne

Certaines données restent disponibles :

- historique local ;
- cartes ;
- QR ;
- profil ;
- favoris ;
- préférences.

Les opérations sont synchronisées au retour du réseau.

## 19. Animations

Toutes les animations doivent fonctionner à 60 FPS lorsque réaliste.

Technologies :

- Framer Motion ;
- GSAP ;
- React Native Reanimated.

Effets :

- fade ;
- slide ;
- scale ;
- micro-interactions ;
- haptics ;
- skeleton loading ;
- glassmorphism.

## 20. Administration

Le back-office peut :

- ajouter des widgets ;
- retirer des widgets ;
- changer leur ordre ;
- imposer des widgets ;
- masquer des fonctionnalités selon le pays ;
- personnaliser les raccourcis ;
- afficher une bannière ;
- pousser une campagne.

## 21. Contrats API

```http
GET /dashboard
GET /dashboard/widgets
PATCH /dashboard/layout
GET /dashboard/activity
GET /dashboard/cards
GET /dashboard/accounts
GET /dashboard/promotions
GET /dashboard/jini
GET /dashboard/alerts
GET /dashboard/news
PATCH /dashboard/preferences
GET /dashboard/search
POST /dashboard/widget
```

## 22. Modèles

- Dashboard ;
- DashboardWidget ;
- QuickAction ;
- DashboardLayout ;
- DashboardCard ;
- DashboardAccount ;
- DashboardAlert ;
- DashboardActivity ;
- DashboardBanner ;
- DashboardPreference ;
- DashboardRecommendation ;
- DashboardAnalytics.

## 23. Règles métier

- Le solde masqué reste masqué partout.
- Les widgets suivent l'utilisateur sur tous ses appareils.
- Les widgets imposés ne peuvent pas être supprimés.
- Les actions rapides sont configurables.
- Les alertes critiques restent toujours visibles.
- L'administration peut désactiver un widget par pays.
- Les animations respectent « Réduire les animations » du système.
- Les données sont mises en cache intelligemment.
- L'accueil fonctionne en connexion lente.

## 24. Analytics

- dashboard_open ;
- dashboard_close ;
- widget_open ;
- widget_move ;
- widget_delete ;
- quick_action_used ;
- search_used ;
- dashboard_refresh ;
- jini_clicked ;
- promotion_clicked.

## 25. Critères d'acceptation

Le module est validé lorsque :

- l'accueil charge en moins de deux secondes ;
- les widgets sont personnalisables ;
- les cartes fonctionnent ;
- les comptes sont synchronisés ;
- les actions rapides sont configurables ;
- les alertes apparaissent correctement ;
- Jini fonctionne ;
- le mode hors ligne est opérationnel ;
- les animations restent fluides.

## 26. Tests

- tests unitaires ;
- tests d'intégration ;
- tests End-to-End ;
- tests de performance ;
- tests hors ligne ;
- tests d'accessibilité ;
- tests multi-appareils ;
- tests de synchronisation ;
- tests de personnalisation.
