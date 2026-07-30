# Volume 1 — Architecture cible

## 1. Vue d’ensemble

Mansa est construit comme un monorepo modulaire. Le cœur métier reste indépendant des interfaces mobiles, web et des fournisseurs externes. Les premières livraisons peuvent fonctionner comme un monolithe modulaire, puis certains domaines peuvent être extraits en services lorsque la charge, l’organisation ou les contraintes réglementaires le justifient.

## 2. Composants principaux

### Interfaces

- Application Client.
- Application Commerçant.
- Application TPE Android.
- Application Admin Lite.
- Application Annuaire.
- Portail administrateur web.
- Site public.

### Backend

- API Gateway NestJS.
- Modules métier isolés : identité, KYC, comptes, grand livre, paiements, cartes, commerçants, administration, services publics, support et notifications.
- Travailleurs asynchrones pour webhooks, rapprochements, notifications, exports et traitements longs.
- Adaptateurs dédiés pour banques, Mobile Money, cartes, SMS, e-mail et identité.

### Données

- PostgreSQL comme base transactionnelle.
- Grand livre en partie double pour tous les mouvements financiers.
- Redis pour cache, verrous courts, limitation de débit et files lorsque configuré.
- Stockage objet pour justificatifs et exports, avec chiffrement et durées de conservation.
- Entrepôt analytique séparé à terme afin de ne pas perturber les transactions.

## 3. Découpage du monorepo

```text
apps/
  api-gateway/
  admin-web/
  public-web/
  mobile-client/
  mobile-merchant/
  mobile-admin-lite/
  mobile-directory/
  tpe-android/
packages/
  config/
  contracts/
  domain/
  ui/
```

Les applications ne partagent pas directement leurs couches d’accès aux données. Les éléments partagés sont limités aux contrats, primitives métier, configurations et composants d’interface génériques.

## 4. Flux transactionnel de référence

1. Le client envoie une requête avec un identifiant d’idempotence.
2. L’API authentifie l’acteur et vérifie ses autorisations, limites et état KYC.
3. Le module métier crée une intention de transaction.
4. Le grand livre réserve ou débite les fonds dans une transaction de base atomique.
5. L’adaptateur externe est appelé lorsque nécessaire.
6. Le résultat est enregistré et un événement de domaine est publié.
7. Les notifications, webhooks et rapprochements sont traités de façon asynchrone.
8. Toute transition est inscrite dans les journaux d’audit et d’événements.

## 5. Sécurité architecturale

- Authentification forte pour les opérations sensibles.
- RBAC complété par des règles contextuelles.
- Chiffrement TLS en transit et chiffrement au repos.
- Isolation stricte entre environnements et partenaires.
- Validation des entrées à toutes les frontières.
- Rotation des secrets et clés hors dépôt.
- Journalisation sans données sensibles complètes.
- Défense contre rejeu grâce à l’idempotence, aux nonces et aux horodatages.

## 6. Résilience

- Délais d’expiration explicites pour tous les appels externes.
- Réessais bornés avec temporisation et file d’échec.
- Coupe-circuit par partenaire.
- Opérations compensatoires plutôt que modifications silencieuses.
- Sauvegardes testées et objectifs RPO/RTO documentés avant production.
- Mode dégradé explicite pour les terminaux et partenaires indisponibles.

## 7. Contrats et versionnement

- API publique sous préfixe versionné, par exemple `/v1`.
- Schémas OpenAPI générés et contrôlés en CI.
- Événements versionnés et rétrocompatibles.
- Toute rupture de contrat exige une nouvelle version et un plan de migration.

## 8. Critères de validation du socle

- Installation reproductible depuis un clone propre.
- Lint, vérification TypeScript, tests et build exécutables à la racine.
- Aucun secret détecté dans l’historique livré.
- Modules backend indépendants et testables.
- Documentation cohérente avec les chemins et scripts réels du monorepo.
