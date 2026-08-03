# Porte de décision transactionnelle

## 1. Objet

La porte transactionnelle consolide les contrôles de risque, d’appareil, de limites et de bénéficiaire avant l’exécution d’une opération financière.

Le contrat correspondant est `packages/security/src/transaction-gate.ts` dans `mansa-platform`.

Cette couche ne remplace ni l’autorisation RBAC/ABAC, ni le grand livre, ni l’idempotence. Elle intervient après l’identification de l’acteur et avant la création de l’ordre financier définitif.

## 2. Entrées obligatoires

L’évaluateur reçoit quatre résultats déjà calculés :

- évaluation du risque transactionnel ;
- évaluation de confiance de l’appareil ;
- résultat de la politique de limites ;
- résultat de la politique de bénéficiaire.

Les intégrations ne doivent pas fabriquer directement une décision finale. Chaque domaine conserve son évaluateur spécialisé puis transmet son résultat à la porte commune.

## 3. Décisions

La porte renvoie exactement une décision :

- `ALLOW` : tous les contrôles autorisent l’opération ;
- `REQUIRE_STEP_UP` : une authentification renforcée récente est nécessaire ;
- `REQUIRE_REVIEW` : une revue humaine, une réinscription de l’appareil ou une analyse risque est nécessaire ;
- `BLOCK` : l’opération ne doit pas être créée.

Chaque décision contient une liste de raisons stables destinée aux journaux d’audit, métriques, alertes et interfaces.

## 4. Ordre de priorité

L’ordre est volontairement strict :

1. un refus de limite, de bénéficiaire, de risque ou d’intégrité appareil produit `BLOCK` ;
2. une demande de revue ou de réinscription produit `REQUIRE_REVIEW` ;
3. une demande d’authentification renforcée produit `REQUIRE_STEP_UP` ;
4. l’absence de contrainte produit `ALLOW`.

Une décision moins stricte ne peut jamais masquer une décision plus stricte. Ainsi, une revue appareil reste prioritaire même lorsque le risque demande seulement une élévation d’authentification.

## 5. Raisons bloquantes

Les raisons minimales sont :

- `LIMIT_REJECTED` ;
- `BENEFICIARY_REJECTED` ;
- `RISK_BLOCKED` ;
- `DENY_BLOCKED_DEVICE` ;
- `DENY_COMPROMISED_DEVICE` ;
- `DENY_INTEGRITY_FAILURE`.

Toutes les causes détectées sont conservées. Cette agrégation évite de perdre un signal de fraude lorsqu’une autre règle a déjà bloqué la transaction.

## 6. Intégration attendue

Le cas d’usage de paiement doit suivre cet ordre :

1. authentifier la session ;
2. autoriser la permission et la portée ;
3. charger les règles versionnées applicables ;
4. évaluer limites, bénéficiaire, appareil et risque ;
5. appeler `evaluateTransactionGate` ;
6. journaliser la décision et les versions de politiques ;
7. exécuter seulement si la décision est `ALLOW` ;
8. déclencher le parcours renforcé ou la revue pour les autres décisions.

Aucune écriture irréversible dans le grand livre ne doit précéder cette décision.

## 7. Tests automatiques

La suite `packages/security/src/transaction-gate.test.ts` vérifie :

- le chemin nominal autorisé ;
- l’agrégation de plusieurs causes bloquantes ;
- la priorité de la revue sur l’authentification renforcée ;
- la combinaison des raisons de step-up appareil et risque.

## 8. Commandes de validation

```bash
pnpm --filter @mansa/security build
pnpm --filter @mansa/security typecheck
pnpm --filter @mansa/security test
```

## 9. Critères d’acceptation

- Une limite refusée bloque toujours l’opération.
- Un bénéficiaire refusé bloque toujours l’opération.
- Un appareil compromis, bloqué ou non intègre bloque toujours l’opération.
- Une décision risque `BLOCK` ne peut être dégradée.
- Une revue est prioritaire sur un step-up.
- Toutes les raisons bloquantes détectées sont conservées.
- Le contrat est exporté par `packages/security/src/index.ts`.
- Les applications n’implémentent pas leur propre ordre de priorité divergent.
