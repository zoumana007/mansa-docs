# Transferts, bénéficiaires et demandes d’argent

## 1. Objectif

Mansa Client doit permettre d’envoyer, recevoir et demander de l’argent de manière simple, traçable et sûre. L’expérience utilisateur ne doit jamais masquer les contrôles exécutés par le backend : identité du destinataire, disponibilité des fonds, limites, frais, conformité, fraude et idempotence.

## 2. Canaux de transfert

Les transferts peuvent être initiés vers :

- un identifiant Mansa ;
- un numéro de téléphone vérifié ;
- un QR Mansa ;
- un portefeuille interne ;
- un commerçant ;
- un compte bancaire partenaire ;
- un compte Mobile Money partenaire ;
- un bénéficiaire enregistré ;
- une demande d’argent reçue ;
- un lien de paiement ou de collecte.

Chaque canal est activé par pays, devise, partenaire, segment client, niveau KYC et environnement.

## 3. Recherche du destinataire

Avant la saisie du montant, l’application peut rechercher un destinataire à partir de son identifiant Mansa, numéro de téléphone ou QR.

Le backend retourne uniquement les informations nécessaires à la confirmation :

- nom d’affichage ou nom masqué ;
- avatar autorisé ;
- identifiant Mansa ;
- type de destinataire ;
- pays ;
- canal disponible ;
- avertissement éventuel.

Les données personnelles complètes, documents KYC et coordonnées non nécessaires ne sont jamais exposés.

## 4. Confirmation forte du destinataire

Avant validation, l’écran affiche clairement :

- le destinataire ;
- le canal ;
- le montant ;
- la devise ;
- les frais ;
- le total débité ;
- le montant estimé reçu ;
- le taux de change lorsqu’il existe ;
- la référence ou le motif ;
- le délai estimé ;
- les mentions de non-réversibilité lorsqu’elles s’appliquent.

Un changement de destinataire, montant, devise, frais ou taux invalide la confirmation précédente.

## 5. Commande de transfert

Le backend reçoit une commande immuable contenant au minimum :

- un identifiant unique de transfert ;
- le portefeuille source ;
- le portefeuille ou canal destination ;
- un montant strictement positif ;
- une devise ;
- une clé d’idempotence ;
- une référence facultative ;
- le contexte d’authentification ;
- le canal d’origine ;
- les métadonnées techniques autorisées.

Le portefeuille source et le portefeuille destination doivent être différents.

## 6. Idempotence

La répétition de la même requête avec la même clé d’idempotence ne doit jamais créer un second débit.

Le backend doit :

1. détecter une commande déjà traitée ;
2. retourner le résultat initial ;
3. refuser la réutilisation de la clé avec un contenu différent ;
4. conserver une durée de rétention configurable ;
5. tracer les collisions et anomalies.

L’application mobile génère une clé stable pour une tentative logique et ne la renouvelle qu’après abandon explicite ou nouvelle opération.

## 7. Vérifications avant exécution

Le backend vérifie au minimum :

1. l’authentification et l’autorisation ;
2. le statut du client et du portefeuille ;
3. le niveau KYC ;
4. le statut du destinataire ;
5. la validité du canal ;
6. la devise ;
7. le montant minimal et maximal ;
8. le solde disponible ;
9. les limites transactionnelles ;
10. les frais applicables ;
11. les sanctions et listes de surveillance ;
12. les règles de fraude ;
13. l’idempotence ;
14. la disponibilité du partenaire externe.

Aucune décision financière finale n’est prise uniquement dans l’application mobile.

## 8. Frais et taux de change

Les frais peuvent être :

- fixes ;
- proportionnels ;
- combinés ;
- plafonnés ;
- pris en charge par l’émetteur ;
- déduits du montant reçu ;
- subventionnés par un partenaire ;
- gratuits selon une promotion ou un quota.

Le devis présenté doit posséder une date d’expiration. Une modification des frais ou du taux impose une nouvelle confirmation.

## 9. Exécution comptable

Un transfert interne réussi doit produire une écriture équilibrée dans le grand livre. Le débit, le crédit, les frais et taxes éventuels sont enregistrés atomiquement ou compensés selon une procédure explicite.

Pour un transfert externe, le système distingue :

- l’acceptation de la demande ;
- la réservation des fonds ;
- l’envoi au partenaire ;
- l’accusé de réception ;
- la confirmation finale ;
- l’échec ;
- l’expiration ;
- la compensation ou le remboursement.

## 10. Statuts utilisateur

Les statuts affichés sont harmonisés avec le backend :

- initié ;
- en validation ;
- en attente ;
- envoyé au partenaire ;
- réussi ;
- échoué ;
- annulé ;
- expiré ;
- remboursé ;
- contesté.

Le statut « réussi » n’est affiché qu’après confirmation définitive de la source d’autorité.

## 11. Bénéficiaires enregistrés

L’utilisateur peut enregistrer un bénéficiaire avec :

- un alias personnel ;
- le canal ;
- l’identifiant du destinataire ;
- la devise habituelle ;
- une note facultative ;
- une date de dernière utilisation ;
- un statut actif ou bloqué.

L’ajout ou la modification d’un bénéficiaire sensible peut exiger une authentification forte et un délai de sécurité configurable.

## 12. Protection contre l’erreur et la fraude

Le système doit prévoir :

- alerte lors d’un premier transfert ;
- avertissement pour un nouveau bénéficiaire ;
- détection d’un montant inhabituel ;
- contrôle des changements récents de téléphone ou appareil ;
- confirmation renforcée pour les opérations à risque ;
- blocage temporaire après plusieurs échecs ;
- message anti-arnaque avant certains transferts ;
- possibilité d’interrompre avant exécution finale.

Les motifs internes de fraude ne sont pas révélés dans l’interface.

## 13. Demandes d’argent

Un utilisateur peut créer une demande contenant :

- un demandeur ;
- un ou plusieurs payeurs ;
- un montant fixe ou libre ;
- une devise ;
- une description ;
- une date d’expiration ;
- un partage égal ou personnalisé ;
- un lien ou QR ;
- un statut.

Le payeur peut accepter, refuser, payer partiellement lorsque cela est autorisé ou signaler la demande.

## 14. Partage de dépense

Le partage de dépense permet :

- la division égale ;
- la répartition personnalisée ;
- l’exclusion de certains articles ;
- l’ajout facultatif d’un pourboire ;
- le suivi des montants dus et payés ;
- les relances configurables ;
- la clôture automatique lorsque le total est atteint.

La somme des parts doit toujours correspondre au montant total en unités mineures. Les écarts d’arrondi sont affectés selon une règle déterministe.

## 15. Annulation et remboursement

Un transfert peut être annulé uniquement avant le point d’irréversibilité défini par le canal.

Après ce point :

- une demande de remboursement distincte est créée ;
- le bénéficiaire ou le partenaire peut devoir l’accepter ;
- toute compensation produit de nouvelles écritures ;
- l’opération originale reste conservée ;
- le lien entre opération originale et remboursement est immuable.

Aucune suppression d’historique financier n’est autorisée.

## 16. Reçus et preuves

Après traitement, l’utilisateur peut consulter ou partager un reçu contenant :

- référence Mansa ;
- statut ;
- date et heure ;
- émetteur masqué ;
- destinataire masqué ;
- montant ;
- devise ;
- frais ;
- canal ;
- référence utilisateur ;
- identifiant partenaire lorsque pertinent.

Le reçu partagé ne doit pas exposer de donnée sensible inutile.

## 17. Notifications

Les notifications peuvent couvrir :

- transfert initié ;
- authentification requise ;
- transfert en attente ;
- transfert réussi ;
- transfert échoué ;
- demande reçue ;
- demande payée ;
- remboursement ;
- activité suspecte.

Le contenu affiché sur écran verrouillé respecte les préférences de confidentialité.

## 18. Administration

L’administration peut configurer :

- canaux disponibles ;
- montants minimums et maximums ;
- frais ;
- taux et marges ;
- délais d’expiration ;
- règles de nouveaux bénéficiaires ;
- authentification forte ;
- partage de dépense ;
- motifs et catégories ;
- messages anti-fraude ;
- maintenance par partenaire ;
- règles par pays, segment et niveau KYC.

Toute modification est versionnée et auditée.

## 19. Contrats techniques existants

Le dépôt `mansa-platform` fournit déjà les bases de domaine suivantes :

- `TransferCommand` pour valider une intention immuable ;
- `TransferService` pour orchestrer le transfert ;
- `TransferRepository` pour la persistance ;
- `TransferResult` pour le résultat ;
- `WalletTransferExecutor` pour le mouvement entre portefeuilles ;
- `TransactionLimitPolicy` pour les limites ;
- `Money` pour les montants en unités mineures ;
- les contrats d’idempotence, de transaction et d’événements.

Les futurs modules API doivent réutiliser ces contrats plutôt que dupliquer les règles dans les contrôleurs.

## 20. Critères de recette

- Un montant nul ou négatif est refusé.
- La source et la destination ne peuvent pas être identiques.
- Une même clé d’idempotence ne produit jamais deux débits.
- Une clé réutilisée avec une commande différente est refusée.
- Le destinataire est confirmé avant validation.
- Les frais et le total débité sont visibles avant authentification finale.
- Un transfert supérieur aux limites est refusé.
- Un solde insuffisant ne crée aucune écriture partielle.
- Un transfert interne réussi est comptablement équilibré.
- Un transfert externe ne devient réussi qu’après confirmation partenaire.
- Les bénéficiaires sensibles exigent les contrôles configurés.
- Une demande expirée ne peut plus être payée.
- La somme des parts d’une dépense correspond exactement au total.
- Les remboursements conservent un lien immuable avec l’opération originale.
- Toutes les transitions et décisions sensibles sont auditées.
