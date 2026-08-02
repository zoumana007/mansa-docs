# Matrice de recette transverse

## Objet

Cette matrice définit les contrôles communs à tous les produits Mansa. Elle complète les critères propres à chaque volume et sert de référence minimale avant toute promotion vers Recette ou Production.

## Règles d’utilisation

- Chaque scénario possède un identifiant stable.
- Une preuve est jointe à chaque exécution : journal, capture, rapport automatisé ou référence de transaction de test.
- Un échec bloquant interdit la promotion de l’environnement.
- Les jeux de données utilisent uniquement des identités fictives.
- Les tests financiers utilisent les unités mineures et vérifient l’équilibre comptable.

## Identité, authentification et sessions

| ID | Scénario | Résultat attendu | Niveau |
|---|---|---|---|
| REC-AUTH-001 | Connexion avec identifiants valides | Session créée, jetons limités dans le temps, événement d’audit enregistré | Bloquant |
| REC-AUTH-002 | Mot de passe erroné répété | Limitation progressive, alerte selon politique, aucune information sensible révélée | Bloquant |
| REC-AUTH-003 | Révocation d’une session | Jeton de renouvellement inutilisable et appareils concernés déconnectés | Bloquant |
| REC-AUTH-004 | Action sensible sans authentification renforcée | Action refusée avec code d’erreur documenté | Bloquant |
| REC-AUTH-005 | Changement de rôle pendant une session | Nouvelles permissions appliquées sans attendre l’expiration complète de la session | Bloquant |

## Autorisations et administration

| ID | Scénario | Résultat attendu | Niveau |
|---|---|---|---|
| REC-AUTHZ-001 | Utilisateur sans permission appelle une route protégée | Réponse interdite, aucun effet métier, audit de refus disponible | Bloquant |
| REC-AUTHZ-002 | Modification d’une configuration critique | Double validation appliquée lorsque requise et historique conservé | Bloquant |
| REC-AUTHZ-003 | Désactivation d’un produit ou partenaire | Nouvelle opération bloquée immédiatement, opérations déjà engagées traitées selon politique | Bloquant |
| REC-AUTHZ-004 | Export administratif | Périmètre limité au rôle et données sensibles masquées | Majeur |

## Comptes, portefeuille et ledger

| ID | Scénario | Résultat attendu | Niveau |
|---|---|---|---|
| REC-LEDGER-001 | Écriture d’une transaction financière | Total débits égal au total crédits | Bloquant |
| REC-LEDGER-002 | Rejeu avec la même clé d’idempotence | Aucun doublon, réponse cohérente avec la première exécution | Bloquant |
| REC-LEDGER-003 | Solde insuffisant | Transaction refusée sans écriture partielle | Bloquant |
| REC-LEDGER-004 | Annulation ou remboursement | Écritures compensatoires créées, historique initial conservé | Bloquant |
| REC-LEDGER-005 | Panne après réservation mais avant confirmation | État récupérable automatiquement ou par procédure documentée | Bloquant |
| REC-LEDGER-006 | Rapprochement quotidien | Écarts identifiés, quantifiés et exportables | Majeur |

## Paiements, transferts et intégrations

| ID | Scénario | Résultat attendu | Niveau |
|---|---|---|---|
| REC-PAY-001 | Paiement réussi | Statut final, reçu, écritures et notifications cohérents | Bloquant |
| REC-PAY-002 | Délai d’attente partenaire | Statut intermédiaire explicite, aucun débit double, réconciliation possible | Bloquant |
| REC-PAY-003 | Webhook dupliqué | Un seul effet métier, chaque réception journalisée | Bloquant |
| REC-PAY-004 | Signature de webhook invalide | Rejet sans traitement métier et alerte de sécurité | Bloquant |
| REC-PAY-005 | Partenaire indisponible | Circuit de protection actif, message utilisateur compréhensible, reprise contrôlée | Majeur |
| REC-PAY-006 | Conversion de devise | Taux, frais, arrondis et devise de règlement conservés dans la transaction | Bloquant |

## KYC, conformité et risque

| ID | Scénario | Résultat attendu | Niveau |
|---|---|---|---|
| REC-KYC-001 | Inscription avec niveau KYC incomplet | Limites adaptées et fonctions réglementées indisponibles | Bloquant |
| REC-KYC-002 | Document expiré | Statut mis à jour et opérations concernées limitées selon politique | Bloquant |
| REC-KYC-003 | Détection d’un seuil ou comportement à risque | Alerte créée sans bloquer abusivement les fonctions non concernées | Majeur |
| REC-KYC-004 | Consultation d’une donnée KYC | Accès réservé, motif et acteur audités | Bloquant |
| REC-KYC-005 | Suppression ou anonymisation réglementaire | Politique de conservation respectée sans casser les obligations comptables | Bloquant |

## Applications et expérience utilisateur

| ID | Scénario | Résultat attendu | Niveau |
|---|---|---|---|
| REC-UX-001 | Perte réseau pendant une opération | Aucun doublon et état final récupéré après reconnexion | Bloquant |
| REC-UX-002 | Version d’application non supportée | Mise à jour imposée ou recommandée selon configuration | Majeur |
| REC-UX-003 | Langue et devise changées | Libellés, formats et montants cohérents sans altérer les données | Majeur |
| REC-UX-004 | Accessibilité des parcours essentiels | Navigation clavier/lecteur d’écran ou équivalent vérifiée selon plateforme | Majeur |
| REC-UX-005 | Mode hors ligne autorisé | Uniquement les actions prévues sont disponibles et synchronisées de façon idempotente | Bloquant |

## Notifications et support

| ID | Scénario | Résultat attendu | Niveau |
|---|---|---|---|
| REC-NOTIF-001 | Notification transactionnelle | Contenu sans secret, bon destinataire, corrélation avec l’événement métier | Majeur |
| REC-NOTIF-002 | Échec d’un canal | Nouvelle tentative contrôlée ou bascule selon politique | Majeur |
| REC-SUP-001 | Création d’un ticket | Référence unique, historique immuable des changements, SLA calculable | Majeur |
| REC-SUP-002 | Agent support consulte un compte | Accès limité, données masquées et action auditée | Bloquant |

## Sécurité, confidentialité et audit

| ID | Scénario | Résultat attendu | Niveau |
|---|---|---|---|
| REC-SEC-001 | Recherche de secret dans le dépôt et les artefacts | Aucun secret réel détecté | Bloquant |
| REC-SEC-002 | Entrée malveillante courante | Validation, échappement et journalisation sans exposition interne | Bloquant |
| REC-SEC-003 | Téléversement de fichier interdit | Type, taille et contenu refusés avant stockage permanent | Bloquant |
| REC-SEC-004 | Lecture d’un journal d’audit | Acteur, action, cible, date, résultat et corrélation présents | Bloquant |
| REC-SEC-005 | Tentative d’altération d’un audit | Refus ou détection vérifiable | Bloquant |
| REC-SEC-006 | Export de données personnelles | Consentement ou base légale, périmètre et traçabilité vérifiés | Bloquant |

## Performance, disponibilité et reprise

| ID | Scénario | Résultat attendu | Niveau |
|---|---|---|---|
| REC-OPS-001 | Charge nominale définie | Latence et taux d’erreur respectent les objectifs du service | Majeur |
| REC-OPS-002 | Pic de charge | Dégradation contrôlée, aucune corruption financière | Bloquant |
| REC-OPS-003 | Redémarrage d’un service | Reprise sans perte des événements persistés | Bloquant |
| REC-OPS-004 | Restauration d’une sauvegarde | Procédure exécutée, intégrité et point de reprise vérifiés | Bloquant |
| REC-OPS-005 | Indisponibilité d’une zone ou dépendance critique | Bascule ou mode dégradé conforme au plan de continuité | Bloquant |
| REC-OPS-006 | Rotation d’un secret | Aucun secret versionné, service restauré sans changement de code | Bloquant |

## Observabilité

| ID | Scénario | Résultat attendu | Niveau |
|---|---|---|---|
| REC-OBS-001 | Suivi d’une transaction de bout en bout | Identifiant de corrélation commun aux services, workers et journaux | Majeur |
| REC-OBS-002 | Erreur applicative | Journal structuré sans donnée sensible et métrique incrémentée | Majeur |
| REC-OBS-003 | Seuil d’alerte dépassé | Alerte reçue avec service, environnement et procédure associée | Majeur |

## Conditions de passage en production

La promotion en Production exige au minimum :

1. Tous les scénarios bloquants applicables réussis.
2. Aucun défaut critique ou majeur non accepté formellement.
3. Rapport de vulnérabilités examiné.
4. Sauvegarde et restauration testées sur un environnement représentatif.
5. Rapprochement financier validé.
6. Matrice des rôles approuvée.
7. Procédures d’incident, retour arrière et communication disponibles.
8. Paramètres de Production revus par deux personnes autorisées.
9. Contrats et autorisations des partenaires concernés confirmés.
10. Décision de mise en production horodatée et auditée.
