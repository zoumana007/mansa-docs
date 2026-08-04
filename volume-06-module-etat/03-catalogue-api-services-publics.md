# Catalogue API — Services publics

## 1. Objet

Ce catalogue décrit les routes de référence du module État pour les organismes publics, obligations administratives, encaissements, bourses et cartes étudiantes. Les contrats TypeScript correspondants sont maintenus dans `mansa-platform/packages/contracts/src/public-services-api.ts`.

Toutes les routes sont préfixées par `/v1/public-services`. Elles appliquent les conventions communes d’authentification, autorisation, corrélation, pagination, erreurs et idempotence du volume 1.

## 2. Organismes et catalogue de services

| Opération | Méthode | Route | Résultat |
| --- | --- | --- | --- |
| Lister les organismes | `GET` | `/v1/public-services/organizations` | page de `PublicOrganization` |
| Lister les services disponibles | `GET` | `/v1/public-services/catalog` | page de `PublicServiceCatalogEntry` |

Les filtres d’organismes couvrent le pays, la zone administrative et l’état actif. Le catalogue peut être filtré par organisme, type d’obligation et état actif.

## 3. Obligations et paiements

| Opération | Méthode | Route | Corps ou paramètres | Résultat |
| --- | --- | --- | --- | --- |
| Créer une obligation | `POST` | `/v1/public-services/obligations` | `CreatePublicObligationCommand` | `PublicObligation` |
| Lister les obligations | `GET` | `/v1/public-services/obligations` | filtres et pagination | page de `PublicObligation` |
| Lire une obligation | `GET` | `/v1/public-services/obligations/:obligationId` | `obligationId` | `PublicObligation` |
| Encaisser une obligation | `POST` | `/v1/public-services/obligations/:obligationId/payments` | `CollectPublicPaymentCommand` | `PublicPaymentReceipt` |
| Lire un reçu | `GET` | `/v1/public-services/receipts/:receiptId` | `receiptId` | `PublicPaymentReceipt` |

La création et l’encaissement exigent une clé d’idempotence. Une obligation ne peut être encaissée que dans un état autorisé et pour un montant conforme au solde exigible. Le reçu doit conserver les références de l’organisme, de l’agent, du payeur, de la transaction et de la politique tarifaire appliquée.

Les filtres de recherche incluent au minimum l’organisme, le code service, l’utilisateur concerné, la référence externe et le statut.

## 4. Bourses

| Opération | Méthode | Route | Corps ou paramètres | Résultat |
| --- | --- | --- | --- | --- |
| Lister les demandes | `GET` | `/v1/public-services/scholarships` | filtres et pagination | page de `ScholarshipApplication` |
| Décider une demande | `POST` | `/v1/public-services/scholarships/:applicationId/decision` | `DecideScholarshipCommand` | `ScholarshipApplication` |

Les filtres couvrent l’organisme, le programme, l’année académique, le bénéficiaire et le statut. Toute décision doit être auditée avec l’acteur, le motif, l’horodatage et, lorsque la politique le demande, une double validation.

## 5. Cartes étudiantes

| Opération | Méthode | Route | Corps ou paramètres | Résultat |
| --- | --- | --- | --- | --- |
| Émettre une carte | `POST` | `/v1/public-services/student-cards` | `IssueStudentCardCommand` | `StudentCard` |
| Lister les cartes | `GET` | `/v1/public-services/student-cards` | filtres et pagination | page de `StudentCard` |
| Lire une carte | `GET` | `/v1/public-services/student-cards/:cardId` | `cardId` | `StudentCard` |

Les filtres couvrent l’organisme, l’utilisateur étudiant, la référence étudiante externe, l’année académique et le statut. L’émission doit empêcher les doublons actifs pour un même étudiant, organisme et cycle académique selon la politique configurée.

## 6. Autorisations minimales

- La consultation publique du catalogue peut être ouverte sans exposer de données personnelles.
- La création d’obligations est réservée aux agents habilités de l’organisme concerné.
- L’encaissement exige une identité d’agent ou de canal vérifiée et une autorisation explicite.
- La décision de bourse et l’émission de carte étudiante utilisent des permissions distinctes.
- Les administrateurs Mansa ne doivent pas modifier une décision publique sans délégation contractuelle et trace d’audit.

## 7. Erreurs métier attendues

Les erreurs utilisent `ApiErrorResponse`. Les cas principaux sont : ressource absente, obligation déjà payée ou annulée, montant invalide, référence externe dupliquée, transition de statut interdite, décision déjà finale, carte déjà active, permission insuffisante et partenaire public indisponible.

Les détails ne doivent contenir ni document étudiant, ni pièce KYC, ni secret de partenaire.

## 8. Critères de recette

1. Les réponses paginées utilisent le contrat commun `PageResponse`.
2. Une même création rejouée avec la même clé retourne la même ressource sans doublon.
3. Une clé réutilisée avec une charge différente retourne `IDEMPOTENCY_CONFLICT`.
4. Deux encaissements concurrents ne peuvent pas solder deux fois la même obligation.
5. Toute décision de bourse et émission de carte produit un événement d’audit corrélé.
6. Les filtres d’un organisme ne permettent jamais d’accéder aux données d’un autre organisme sans permission transverse.
7. Les reçus restent consultables selon la politique de conservation même après la désactivation du service.
8. Les contrats documentés correspondent aux constantes `PUBLIC_SERVICES_API_ROUTES` et `PUBLIC_SERVICES_API_METHODS` du monorepo.
