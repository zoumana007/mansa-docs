# Volume 2 — Portefeuilles, soldes et transferts

## 1. Objectif

Le module portefeuille permet au client de consulter ses fonds disponibles, d’identifier les montants bloqués ou en attente et d’effectuer des transferts internes de manière sûre, traçable et idempotente.

## 2. Modèle de portefeuille

Un utilisateur peut posséder plusieurs portefeuilles selon le pays, la devise, le produit ou le partenaire. Chaque portefeuille possède :

- un identifiant stable ;
- un propriétaire ;
- une devise unique ;
- un statut opérationnel ;
- un solde disponible ;
- un solde réservé ;
- un niveau KYC et des limites applicables ;
- des horodatages de création et de mise à jour.

Les montants sont représentés en unités mineures entières. Aucun calcul financier ne doit utiliser de nombre flottant.

## 3. Soldes

Le solde affiché au client est dérivé du grand livre. Le portefeuille expose au minimum :

- `available` : montant immédiatement utilisable ;
- `reserved` : montant bloqué pour une opération en cours ;
- `total` : somme des montants disponible et réservé ;
- `asOf` : instant de référence du calcul.

Une valeur mise en cache ne constitue jamais la source comptable de vérité.

## 4. Statuts du portefeuille

- `ACTIVE` : opérations autorisées dans les limites applicables.
- `RESTRICTED` : consultation autorisée, certaines opérations bloquées.
- `SUSPENDED` : aucune sortie de fonds ; traitement manuel requis.
- `CLOSED` : portefeuille définitivement fermé après apurement.

Tout changement de statut doit être motivé et audité.

## 5. Transfert interne

Le transfert interne déplace une valeur entre deux portefeuilles Mansa de même devise.

Flux de référence :

1. réception d’une commande avec clé d’idempotence ;
2. authentification du client et validation du portefeuille source ;
3. validation du bénéficiaire, de la devise, des limites et du KYC ;
4. réservation atomique du montant et des frais éventuels ;
5. écriture équilibrée dans le grand livre ;
6. confirmation du transfert ;
7. émission d’un événement métier ;
8. notification des parties et mise à disposition du reçu.

## 6. Règles métier

- Le montant doit être strictement positif.
- Les portefeuilles source et destination doivent être distincts.
- Les deux portefeuilles doivent utiliser la même devise.
- Le solde disponible doit couvrir le montant et les frais.
- Les limites journalières, mensuelles et par opération sont évaluées avant réservation.
- Une même clé d’idempotence ne peut produire qu’un seul transfert logique.
- Une annulation comptable crée des écritures compensatoires ; elle ne supprime jamais l’historique.

## 7. Statuts d’un transfert

- `CREATED` : commande acceptée mais fonds non réservés.
- `PENDING` : vérifications ou traitement en cours.
- `COMPLETED` : écritures comptables finalisées.
- `FAILED` : transfert non réalisé, avec motif stable.
- `CANCELLED` : opération abandonnée avant finalisation.
- `REVERSED` : transfert finalisé puis compensé.

## 8. Sécurité et fraude

Les contrôles peuvent inclure : appareil inhabituel, vélocité, bénéficiaire récent, montant anormal, localisation incohérente, compte sous restriction et demande de validation renforcée.

Les données sensibles complètes ne doivent pas être inscrites dans les journaux applicatifs.

## 9. API minimale

- `GET /v1/wallets`
- `GET /v1/wallets/{walletId}`
- `GET /v1/wallets/{walletId}/balance`
- `POST /v1/transfers/internal`
- `GET /v1/transfers/{transferId}`
- `GET /v1/transfers`

Les opérations d’écriture exigent une clé d’idempotence et retournent une référence stable.

## 10. Critères d’acceptation

- Les contrats TypeScript correspondent aux statuts et champs décrits ici.
- Les devises incompatibles sont rejetées avant toute écriture.
- Les doubles soumissions retournent le même résultat logique.
- Les transferts finalisés produisent des écritures équilibrées.
- Les restrictions et limites sont vérifiées côté serveur.
- Chaque transition sensible produit un événement d’audit.
