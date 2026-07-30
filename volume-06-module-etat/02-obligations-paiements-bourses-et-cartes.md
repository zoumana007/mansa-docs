# Volume 6 — Obligations, paiements, bourses et cartes étudiantes

## 1. Catalogue de services

Chaque organisme configure un catalogue versionné de services publics. Une entrée décrit :

- le code du service ;
- le libellé présenté au citoyen ;
- le type d’obligation ;
- la devise et les règles de montant ;
- les canaux de paiement autorisés ;
- les pièces requises ;
- les délais et pénalités ;
- les règles de remboursement ou contestation ;
- les comptes de règlement ;
- la période de validité.

Un tarif publié n’est jamais modifié rétroactivement. Une nouvelle version est créée avec sa date d’effet.

## 2. Cycle d’une obligation

États de référence :

- `DRAFT` : saisie non opposable ;
- `ISSUED` : émise et payable ;
- `PARTIALLY_PAID` : paiement partiel autorisé et commencé ;
- `PAID` : montant exigible entièrement réglé ;
- `OVERDUE` : échéance dépassée ;
- `DISPUTED` : contestation ouverte ;
- `CANCELLED` : annulée avec justification ;
- `EXPIRED` : obligation arrivée à expiration ;
- `REFUNDED` : paiements compensés selon décision.

Toutes les transitions sont validées côté serveur. Un dossier payé ne retourne pas à l’état émis : toute correction passe par remboursement ou écriture compensatoire.

## 3. Encaissement

Les canaux possibles sont activés par organisme et service :

- portefeuille Mansa ;
- Mobile Money ;
- carte bancaire ;
- TPE agréé ;
- virement ou banque partenaire ;
- caisse physique intégrée, lorsque contractuellement autorisée.

Le paiement utilise une clé d’idempotence, une référence d’obligation et une référence de transaction. La confirmation n’est définitive qu’après écriture comptable et confirmation du canal.

## 4. Répartition et règlement

Une obligation peut répartir le montant entre plusieurs bénéficiaires institutionnels. La règle de répartition est figée au moment de l’émission et conservée avec le dossier.

Les règlements aux organismes font l’objet de lots avec :

- période couverte ;
- total brut ;
- frais et commissions ;
- remboursements et ajustements ;
- total net ;
- nombre d’opérations ;
- état de rapprochement ;
- références bancaires.

## 5. Contestations et remboursements

Le citoyen peut ouvrir une contestation selon les délais du service. Le dossier contient le motif, les pièces, les réponses, la décision et les échéances.

Un remboursement :

- référence le paiement initial ;
- possède son propre identifiant d’idempotence ;
- exige une justification ;
- suit une approbation adaptée au montant et au risque ;
- produit une écriture compensatoire ;
- déclenche une notification et un reçu.

## 6. Bourses et aides

Le module de bourse sépare l’éligibilité, l’approbation et le décaissement.

Cycle de référence :

1. import ou dépôt d’une candidature ;
2. vérification d’identité et des critères ;
3. détection des doublons ;
4. instruction ;
5. approbation ;
6. constitution d’un lot de paiement ;
7. contrôle à quatre yeux ;
8. décaissement vers un bénéficiaire vérifié ;
9. rapprochement et traitement des rejets.

Aucun agent seul ne peut créer un bénéficiaire, approuver son dossier et exécuter son paiement.

## 7. Scolarité et inscriptions

Les établissements peuvent publier des frais d’inscription, de réinscription, d’examen, de document ou de service. La référence étudiant et l’année académique sont obligatoires.

Après paiement confirmé, Mansa transmet un événement signé à l’établissement. L’accusé de réception de l’établissement est conservé afin d’éviter qu’un paiement réussi reste non appliqué.

## 8. Carte étudiante

La carte étudiante numérique ou physique est liée à :

- un établissement ;
- un identifiant étudiant ;
- une année ou période de validité ;
- un statut d’inscription ;
- une photographie ou identité vérifiée selon la politique ;
- des droits facultatifs : accès, restauration, transport, bibliothèque ou avantages.

États : `PENDING`, `ACTIVE`, `SUSPENDED`, `EXPIRED`, `REVOKED`, `REPLACED`.

La carte expose un identifiant public ou QR rotatif. Elle ne révèle pas directement les données personnelles complètes. La vérification retourne uniquement les attributs nécessaires au contrôleur.

## 9. Référentiels et imports

Les imports de masse utilisent un format versionné, une prévalidation et un rapport d’erreurs. Ils ne créent pas partiellement des dossiers sans résultat explicite.

Chaque ligne importée possède une clé externe stable afin de rendre les relances idempotentes.

## 10. Reporting

Les tableaux de bord distinguent au minimum :

- obligations émises, encaissées, échues et contestées ;
- encaissements par service et canal ;
- délais de règlement ;
- remboursements et annulations ;
- dossiers traités par unité, sans classement individuel abusif ;
- anomalies de montants, appareils ou volumes ;
- bourses approuvées, payées, rejetées et rapprochées ;
- cartes étudiantes actives, suspendues et expirées.

## 11. Critères de validation

- Catalogue et tarifs versionnés.
- Paiements idempotents et rapprochables.
- Séparation des tâches pour les décaissements publics.
- Répartition financière figée et auditable.
- Carte étudiante vérifiable avec divulgation minimale.
- Imports relançables sans doublons.
