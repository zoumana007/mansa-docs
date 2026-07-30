# Volume 7 — Jini, risque et gouvernance IA

## 1. Rôle de Jini

Jini est l’assistant intégré à l’écosystème Mansa. Il aide l’utilisateur à comprendre ses opérations, trouver une fonction, préparer une demande de support et recevoir des explications personnalisées. Il ne doit jamais inventer un solde, confirmer une transaction non vérifiée ni exécuter seul une opération financière sensible.

## 2. Canaux

Jini peut être exposé dans l’application Client, l’application Commerçant, l’administration et le support. Chaque canal applique un périmètre de données et d’actions différent selon les permissions de l’acteur.

## 3. Données accessibles

L’assistant reçoit uniquement le contexte minimal nécessaire : langue, pays, type de canal, identifiant de session et références explicitement autorisées. Les numéros complets de carte, secrets, codes OTP, mots de passe et documents KYC bruts ne doivent jamais être envoyés au modèle.

## 4. Actions et confirmation

Les actions sont séparées en trois catégories :

- lecture sûre : explication, recherche d’aide, statut déjà disponible ;
- préparation : préremplissage d’un transfert, d’un ticket ou d’un formulaire ;
- action sensible : paiement, changement de limite, blocage, modification KYC ou décision administrative.

Toute action sensible exige une confirmation explicite dans une interface déterministe, puis les contrôles d’authentification et d’autorisation habituels.

## 5. Escalade humaine

Jini crée ou enrichit un ticket de support lorsqu’il détecte une demande non résolue, une contestation, un risque de fraude, une situation urgente ou une faible confiance. La conversation remise à l’agent est résumée et les données sensibles sont masquées.

## 6. Évaluation du risque

Le moteur de risque produit un score, un niveau, une décision et des signaux explicables. Les décisions possibles sont : autoriser, demander une revue, imposer une vérification supplémentaire ou bloquer.

Le modèle ne débite ni ne crédite directement un compte. Il fournit une décision à une politique métier versionnée. Les règles déterministes restent disponibles pour les cas réglementaires et les contrôles critiques.

## 7. Traçabilité

Chaque évaluation conserve : version du modèle, date, signaux utilisés, décision, transaction concernée et identifiant de corrélation. Les données sont conservées selon une politique définie avec conformité et sécurité.

## 8. Gouvernance

- Jeux de données documentés et contrôlés.
- Validation avant mise en production.
- Déploiement progressif par pays ou population.
- Surveillance des dérives, faux positifs et biais.
- Possibilité de retour immédiat à une version précédente.
- Revue humaine pour les décisions contestables.
- Interdiction d’entraîner un modèle externe avec les données clients sans base légale et accord contractuel.

## 9. Sécurité des instructions

Les contenus utilisateur, documents et pages externes sont considérés comme non fiables. Les outils accessibles au modèle utilisent des listes d’autorisation, des schémas stricts et des permissions limitées. Une instruction contenue dans une donnée ne peut pas modifier les règles système.

## 10. Indicateurs

Les indicateurs couvrent : taux de résolution, taux d’escalade, satisfaction, latence, coût par conversation, erreurs factuelles, faux positifs fraude, faux négatifs, décisions annulées et incidents de sécurité.
