# Inventaire produit complet Mansa

Ce document est la liste de contrôle officielle. Aucun agent de développement ne doit supprimer, fusionner ou ignorer un produit parce que son implémentation est encore partielle.

## Produits utilisateurs

1. **Application Client** (`apps/mobile-client`) : inscription, KYC, wallet, cartes, virements, Mobile Money, QR/NFC, budgets, coffres, abonnements, fidélité, factures, reçus, support, notifications et assistant Jini.
2. **Application Commerçant** (`apps/mobile-merchant`) : KYB, encaissement, catalogue, facturation, remboursements, équipe, caisse, stocks légers, promotions, fidélité, mini-site, rapports et support.
3. **Application TPE Android** (`apps/tpe-android`) : paiement carte/QR/NFC, paiement fractionné entre convives ou clients, hors-ligne contrôlé, annulation, remboursement, impression de reçu, rapprochement et gestion matérielle.
4. **Application Admin Lite** (`apps/mobile-admin-lite`) : alertes, validation d’actions, supervision, consultation de tableaux de bord et interventions limitées selon permissions.
5. **Application Annuaire / Hub** (`apps/mobile-directory`) : recherche locale, catégories, géolocalisation, profils professionnels, mini-sites, promotions, abonnements sectoriels, avis, favoris, itinéraires, prise de contact, réservation selon secteur, paiements Mansa, monétisation et statistiques.

## Produits web

6. **Site public officiel Mansa** (`apps/public-web`) : marque, présentation, chiffres clés, sécurité, fonctionnalités, tarifs, partenaires, impact, actualités, carrières, téléchargement, aide, pages légales, SEO, CMS et analytics.
7. **Site professionnel Mansa Business** (`apps/business-web`) : solutions commerçants, TPE, paiements, mini-sites, fidélité, onboarding, devis, démonstrations, tarifs, documentation commerciale, espace partenaire, formulaires et génération de leads.
8. **Portail Admin Web** (`apps/admin-web`) : administration centrale de tous les produits, utilisateurs, commerçants, TPE, transactions, cartes, wallets, KYC/KYB, sécurité, fraude, support, CMS, Annuaire, État, IA, analytics, monitoring, conformité et configuration.

## Plateforme technique

9. **API Gateway / Backend NestJS** (`apps/api-gateway`) : authentification, orchestration, contrats API, RBAC/ABAC, audit et accès aux domaines.
10. **Services IA** (`apps/ai-services`) : Jini, aide, classification support, détection assistée de fraude et recommandations sous contrôle humain.
11. **Workers** (`apps/workers`) : webhooks, notifications, exports, rapprochements, files de tâches et traitements planifiés.
12. **Packages partagés** (`packages/*`) : contrats, domaine, sécurité, observabilité, configuration et design system.
13. **Infrastructure** (`infra/`) : Démo, Recette, Production, observabilité, sauvegardes, reprise, CI/CD et références de secrets externes.

## Domaines métier obligatoires

Identité et accès ; KYC/KYB ; wallets ; ledger en partie double ; paiements ; cartes physiques, virtuelles, temporaires et jetables ; virements ; Mobile Money ; QR ; NFC ; TPE ; remboursements ; litiges ; fraude ; commissions ; tarification ; fidélité ; cashback ; promotions ; budgets ; coffres ; abonnements ; factures ; reçus ; catalogue ; mini-sites ; annuaire ; notifications ; support ; IA/Jini ; analytics ; rapports ; comptabilité ; audit ; CMS ; module État ; taxes ; amendes ; bourses ; cartes étudiantes ; paiements publics ; partenaires ; banques ; multi-pays ; multi-devise ; multi-langue ; feature flags ; conformité ; monitoring ; incidents ; sauvegarde et reprise.

## Règle de couverture documentaire

Chaque produit doit disposer au minimum des sections suivantes : périmètre, personas, rôles, écrans, parcours, règles métier, erreurs, API, événements, modèle de données, administration, sécurité, observabilité, accessibilité, tests et critères d’acceptation.

## Sources manquantes

Quand un détail dépend d’un export DeepSeek non présent dans le dépôt ou l’historique accessible, inscrire exactement : **À compléter depuis l’export DeepSeek**. Ne pas inventer de décision présentée comme définitive.