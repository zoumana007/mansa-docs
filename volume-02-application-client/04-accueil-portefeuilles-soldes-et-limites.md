# Accueil, portefeuilles, soldes et limites

## 1. Objectif

L’écran d’accueil de Mansa Client doit donner une lecture immédiate, fiable et personnalisable de la situation financière de l’utilisateur. Il ne doit jamais confondre solde comptable, solde disponible, montants bloqués et limites réglementaires.

## 2. Composition de l’accueil

L’accueil peut afficher :

- le portefeuille principal ;
- le solde disponible ;
- le solde comptable ;
- les montants réservés ou bloqués ;
- les dernières opérations ;
- les actions rapides ;
- les cartes ;
- les objectifs et coffres ;
- les demandes d’argent ;
- les alertes de sécurité ;
- les obligations KYC ;
- les services publics favoris ;
- les recommandations Jini ;
- les promotions autorisées.

L’ordre des blocs est personnalisable, sauf pour les alertes critiques imposées par la sécurité, la conformité ou l’administration.

## 3. Portefeuilles

Un utilisateur peut posséder plusieurs portefeuilles selon les produits activés :

- portefeuille principal ;
- portefeuille dans une autre devise ;
- portefeuille commerçant ;
- portefeuille jeune ;
- portefeuille étudiant ;
- portefeuille de dépenses ;
- coffre ou objectif ;
- portefeuille lié à un service public ;
- portefeuille temporaire ou promotionnel.

Chaque portefeuille possède au minimum :

- un identifiant interne immuable ;
- un propriétaire ;
- un type ;
- une devise ;
- un statut ;
- un solde comptable ;
- un solde disponible ;
- un montant réservé ;
- des limites applicables ;
- des dates de création et de mise à jour.

## 4. Soldes

### 4.1 Solde comptable

Le solde comptable correspond à la somme des écritures validées du grand livre.

### 4.2 Solde disponible

Le solde disponible correspond au montant réellement utilisable après déduction des réservations, blocages, cautions, opérations en attente et restrictions.

### 4.3 Montants réservés

Un montant peut être réservé pour :

- une autorisation carte ;
- un paiement TPE non capturé ;
- un retrait en cours ;
- un litige ;
- une caution ;
- une opération différée ;
- un contrôle de conformité.

### 4.4 Règle d’affichage

L’interface affiche par défaut le solde disponible. Le détail permet de consulter le solde comptable et les montants réservés. Aucun écran ne doit présenter un montant réservé comme immédiatement dépensable.

## 5. Masquage du solde

L’utilisateur peut masquer les soldes :

- par défaut à l’ouverture ;
- après une période d’inactivité ;
- sur les captures récentes ;
- dans les widgets système ;
- sur les notifications.

La réaffichage peut exiger une biométrie selon les préférences de sécurité.

## 6. Actions rapides

Actions possibles :

- envoyer ;
- demander ;
- payer par QR ;
- scanner ;
- recharger ;
- retirer ;
- consulter les cartes ;
- payer une facture ;
- créer une cagnotte ;
- alimenter un coffre ;
- ouvrir Mansa Connect.

Chaque action doit être masquée ou désactivée lorsque le pays, le niveau KYC, le statut du compte, le partenaire ou les limites ne l’autorisent pas.

## 7. Limites

Les limites peuvent être définies :

- par opération ;
- par jour ;
- par semaine ;
- par mois ;
- par type d’opération ;
- par canal ;
- par devise ;
- par pays ;
- par niveau KYC ;
- par profil de risque ;
- par âge ;
- par partenaire.

La règle la plus restrictive applicable doit prévaloir.

## 8. Compteurs de consommation

Pour chaque limite, le système conserve :

- la période ;
- le montant autorisé ;
- le montant déjà consommé ;
- le montant restant ;
- les opérations prises en compte ;
- la date de réinitialisation ;
- la source de la règle.

Les opérations annulées ou définitivement échouées doivent libérer leur consommation selon une règle métier explicite et auditée.

## 9. Contrôle avant opération

Avant d’autoriser une opération, le backend vérifie au minimum :

1. le statut du compte et du portefeuille ;
2. le niveau KYC ;
3. la devise ;
4. le montant minimal et maximal ;
5. le solde disponible ;
6. les limites par opération et par période ;
7. les restrictions pays et partenaire ;
8. les contrôles fraude et conformité ;
9. l’idempotence de la demande.

L’application mobile ne constitue jamais la source d’autorité pour une limite.

## 10. Messages utilisateur

En cas de refus, l’application doit afficher un motif compréhensible sans révéler les règles internes de fraude.

Exemples :

- Solde disponible insuffisant.
- Montant supérieur à votre limite par opération.
- Limite journalière atteinte.
- Vérification d’identité nécessaire.
- Fonction temporairement indisponible.

Lorsque cela est possible, l’écran indique le montant restant et la date de réinitialisation.

## 11. Historique récent

L’accueil présente un historique court avec :

- type ;
- libellé ;
- contrepartie ;
- montant ;
- devise ;
- date ;
- statut ;
- icône du canal.

Les statuts visibles sont harmonisés avec le backend : initiée, en attente, réussie, échouée, annulée, remboursée, contestée ou expirée.

## 12. Fonctionnement hors ligne

En mode hors ligne :

- le dernier solde connu doit être marqué comme non actualisé ;
- la date de dernière synchronisation est visible ;
- aucune opération financière n’est confirmée localement comme réussie ;
- les actions nécessitant le backend sont mises en attente ou bloquées selon le produit ;
- les données sensibles en cache sont chiffrées.

## 13. Accessibilité et connexion lente

L’accueil doit prendre en charge :

- lecteurs d’écran ;
- tailles de texte dynamiques ;
- contraste suffisant ;
- navigation clavier lorsque pertinente ;
- mode faible consommation ;
- chargement progressif ;
- mode connexion lente sans animations lourdes.

## 14. Administration

L’administration peut configurer :

- widgets disponibles ;
- ordre par défaut ;
- actions rapides ;
- limites par segment ;
- messages d’information ;
- alertes obligatoires ;
- périodes de maintenance ;
- fonctionnalités par pays ;
- règles de visibilité.

Toute modification de limite doit être versionnée, datée et auditée.

## 15. Contrats techniques attendus

Le domaine partagé doit fournir :

- une valeur monétaire en unités mineures ;
- une politique de limites indépendante de l’interface ;
- un calcul du montant restant ;
- une décision explicite autorisée ou refusée ;
- des motifs de refus stables ;
- des contrôles de devise ;
- des tests unitaires sur les frontières exactes.

## 16. Critères de recette

- Le solde disponible est distinct du solde comptable.
- Les montants réservés ne peuvent pas être dépensés.
- Une opération égale à la limite est autorisée.
- Une opération supérieure d’une unité mineure est refusée.
- Les consommations journalières et mensuelles sont vérifiées.
- Une devise différente de celle de la politique est refusée.
- Un montant nul ou négatif est refusé.
- Le mobile ne peut pas contourner une limite imposée par le backend.
- Le détail d’une limite indique le restant et sa prochaine réinitialisation.
- Toute modification administrative est auditée.
