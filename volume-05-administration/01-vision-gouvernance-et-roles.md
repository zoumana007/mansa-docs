# Volume 5 — Administration : vision, gouvernance et rôles

## 1. Mission du portail administrateur

Le portail administrateur pilote l’ensemble de l’écosystème Mansa sans modifier directement les données en base. Il permet de gérer les utilisateurs, commerçants, terminaux, partenaires, produits, pays, commissions, limites, incidents, fonctions activées et opérations nécessitant une intervention humaine.

L’administration existe sous deux formes complémentaires :

- `admin-web` pour les opérations complètes, les contrôles, la configuration et les investigations ;
- `mobile-admin-lite` pour les alertes, validations urgentes, consultations et blocages immédiats.

## 2. Principes de gouvernance

1. Aucun compte administrateur ne possède implicitement tous les droits.
2. Le Super Admin configure les rôles mais les opérations financières sensibles restent soumises à séparation des responsabilités.
3. Toute action critique exige un motif, un identifiant de dossier et, selon son niveau, une validation secondaire.
4. Les droits sont évalués selon le rôle, le pays, l’organisation, l’environnement et le périmètre métier.
5. Les actions sont inscrites dans un journal d’audit non modifiable depuis l’interface.
6. Les données sensibles sont masquées par défaut et leur révélation est auditée.
7. Les comptes administrateurs utilisent une authentification forte et des sessions de courte durée.

## 3. Rôles de référence

### Super Admin plateforme

- Gère les pays, environnements, organisations, rôles et politiques globales.
- Active ou désactive des modules.
- Ne valide pas seul les remboursements, ajustements comptables ou changements de coordonnées bancaires.

### Admin pays

- Gère la configuration du pays attribué.
- Supervise utilisateurs, commerçants, terminaux et partenaires locaux.
- Ne peut pas accéder aux autres pays sans affectation explicite.

### Conformité et KYC

- Examine les dossiers KYC/KYB.
- Demande des compléments, approuve, refuse ou suspend selon la politique.
- Consulte les preuves nécessaires avec masquage adapté.

### Risque et fraude

- Consulte les alertes et signaux de risque.
- Bloque temporairement un compte, une carte, un terminal ou une transaction.
- Escalade les cas nécessitant une décision financière ou réglementaire.

### Opérations financières

- Suit paiements, transferts, règlements et rapprochements.
- Initie des corrections ou remboursements soumis aux règles d’approbation.
- Ne peut pas modifier les journaux du grand livre.

### Support

- Consulte les informations utiles au traitement d’un ticket.
- Réinitialise certains accès et déclenche des procédures contrôlées.
- Ne voit ni secrets, ni données bancaires complètes, ni documents non nécessaires.

### Gestionnaire commerçants et TPE

- Valide l’activation des commerçants et points de vente.
- Affecte, suspend et configure les terminaux.
- Suit la santé du parc et les incidents matériels.

### Auditeur

- Accès en lecture seule aux configurations, historiques et preuves.
- Aucun droit de mutation.

## 4. Modèle d’autorisation

Les autorisations suivent un modèle RBAC enrichi de contraintes contextuelles :

```text
permission = ressource.action
contexte = pays + organisation + environnement + propriétaire + niveau de risque
```

Exemples :

- `merchant.review` limité au Mali et à l’environnement Production ;
- `payment.refund.request` autorisé à l’opérateur, mais `payment.refund.approve` réservé à un second rôle ;
- `feature-flag.update` interdit en Production sans double validation ;
- `audit.export` réservé aux auditeurs et responsables conformité.

## 5. Séparation des responsabilités

Les opérations suivantes utilisent le principe maker-checker :

- remboursement au-dessus d’un seuil configurable ;
- ajustement de solde ou écriture compensatoire ;
- changement de compte de règlement commerçant ;
- modification d’une commission en Production ;
- activation d’un partenaire financier ;
- changement d’une limite globale ;
- export massif de données ;
- désactivation d’un contrôle de risque.

L’initiateur ne peut jamais approuver sa propre demande.

## 6. Critères d’acceptation

- Chaque écran vérifie les permissions côté interface et côté API.
- Les rôles sont configurables sans supprimer les rôles de sécurité minimaux.
- Toute mutation sensible contient motif, auteur, date, contexte et résultat.
- Les sessions administrateur peuvent être révoquées immédiatement.
- Les accès hors périmètre retournent une erreur explicite sans révéler de données.
- Les journaux permettent de reconstituer l’historique complet d’une décision.
