# Validation de la politique de rôles

## 1. Objet

Ce document définit les contrôles automatiques appliqués au contrat `packages/security/src/role-policy.ts` du dépôt `mansa-platform`.

La politique de rôles constitue le socle RBAC. Elle attribue des permissions de base, mais ne remplace jamais les contrôles ABAC, la propriété de la ressource, les limites, le risque, l’état de session ni la double validation.

## 2. Invariants automatiques

La suite `packages/security/src/role-policy.test.ts` vérifie les invariants suivants :

1. chaque rôle déclaré dans `ROLES` possède exactement une entrée dans `ROLE_PERMISSIONS` ;
2. chaque permission affectée appartient au catalogue `PERMISSIONS` ;
3. aucune permission n’est dupliquée dans un rôle ;
4. la fusion de plusieurs rôles supprime les doublons ;
5. l’absence de rôle ne donne aucun droit ;
6. les comptes de service ne reçoivent aucun privilège implicite ;
7. les contraintes de séparation des tâches restent présentes ;
8. les rôles terrain ne peuvent pas administrer leur propre politique.

## 3. Séparation des tâches

Le rôle `FINANCE_OPERATOR` peut initier une écriture d’ajustement avec `ledger.adjustment.initiate`, mais il ne possède pas `ledger.adjustment.approve`.

L’approbation doit être réalisée par un autre acteur autorisé et contrôlée par l’évaluateur d’autorisation afin d’interdire l’auto-approbation.

Toute évolution de la matrice qui accorderait simultanément l’initiation et l’approbation à un rôle opérationnel doit faire l’objet d’une décision d’architecture et d’une revue sécurité explicite.

## 4. Comptes de service

`SERVICE_ACCOUNT` reste vide par défaut. Les services techniques obtiennent leurs droits via une politique dédiée à leur identité, leur environnement et leur finalité.

Aucun service ne doit hériter d’un rôle humain pour contourner cette règle.

## 5. Agents publics

`PUBLIC_AGENT` peut consulter un service public, créer un dossier et collecter un paiement. Il ne peut pas :

- modifier la configuration du service ;
- gérer les agents ;
- annuler seul un dossier finalisé ;
- exporter des données globales.

Ces restrictions protègent la séparation entre exécution terrain, supervision et administration.

## 6. Procédure de modification

Toute modification de `ROLES`, `PERMISSIONS` ou `ROLE_PERMISSIONS` doit :

1. mettre à jour la documentation de la matrice RBAC ;
2. ajouter ou adapter les tests d’invariants ;
3. vérifier les conséquences sur les routes et cas d’usage concernés ;
4. confirmer les règles de périmètre ABAC ;
5. confirmer les obligations d’audit et de double validation ;
6. passer le build, le typecheck et les tests du paquet sécurité.

## 7. Commandes de validation

Depuis la racine du monorepo :

```bash
pnpm --filter @mansa/security build
pnpm --filter @mansa/security typecheck
pnpm --filter @mansa/security test
```

Les tests s’exécutent sur les fichiers JavaScript produits dans `dist` après compilation TypeScript.

## 8. Critères d’acceptation

- Aucun rôle déclaré ne manque dans la politique.
- Aucune permission inconnue n’est affectée.
- Aucun doublon n’est présent dans les droits d’un rôle.
- Un agent public ne peut administrer ni les services ni les agents.
- Un opérateur financier ne peut approuver sa propre catégorie d’ajustement.
- Un compte de service ne reçoit aucun privilège par défaut.
- La documentation et les contrats TypeScript utilisent les mêmes noms de rôles et permissions.
