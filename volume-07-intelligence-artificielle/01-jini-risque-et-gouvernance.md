# Jini, évaluation du risque et gouvernance de l’IA

## Périmètre

Le domaine Intelligence couvre l’assistant Jini et l’évaluation du risque transactionnel. Aucun modèle ne constitue seul l’autorité pour une décision financière, réglementaire ou irréversible : le backend, les règles de conformité, les limites et les autorisations restent déterminants.

## Contrats de référence

- `mansa-platform/packages/contracts/src/intelligence.ts`
- `mansa-platform/packages/contracts/src/intelligence-api.ts`
- `mansa-platform/packages/contracts/src/ai-governance.ts`

## Jini

Jini peut assister les clients, commerçants, administrateurs et équipes support. Il peut expliquer une fonction, guider un parcours, résumer des informations déjà autorisées et préparer un ticket de support.

Jini ne doit jamais révéler un OTP, un secret, un numéro de carte complet, un CVV ou une donnée KYC inutile. Il ne modifie pas directement un solde, une écriture comptable ou une décision de conformité. Toute action sensible passe par le backend, les contrôles RBAC/ABAC et une confirmation adaptée.

## Évaluation du risque

Une évaluation reçoit au minimum la transaction, l’utilisateur, le montant en unité mineure, la devise, le canal et l’horodatage. Elle peut utiliser des signaux approuvés liés à l’appareil, au terminal, à la localisation et au comportement.

Le résultat contient un score entier de `0` à `100`, un niveau `LOW`, `MEDIUM`, `HIGH` ou `CRITICAL`, une décision `ALLOW`, `REVIEW`, `CHALLENGE` ou `BLOCK`, les signaux, la version du modèle et la date d’évaluation.

## Matrice de décision

| Décision | Effet |
|---|---|
| `ALLOW` | Continuer vers les contrôles métier suivants. |
| `REVIEW` | Mettre l’opération en attente pour revue traçable. |
| `CHALLENGE` | Exiger une authentification ou preuve supplémentaire. |
| `BLOCK` | Refuser selon une règle explicite et auditer le motif. |

La décision de risque ne remplace jamais les contrôles de solde, limite, conformité, autorisation ou idempotence.

## Cohérence minimale

- score entier entre `0` et `100` ;
- `LOW` pour `0–29` ;
- `MEDIUM` pour `30–59` ;
- `HIGH` pour `60–84` ;
- `CRITICAL` pour `85–100` ;
- `ALLOW` interdit avec `CRITICAL` ;
- `BLOCK` interdit avec `LOW` ;
- identifiants, version de modèle et horodatage obligatoires ;
- codes de signaux non vides et scores de signaux entre `0` et `100`.

Ces seuils forment le socle initial. Toute variation par pays, produit ou partenaire est versionnée, approuvée et auditée.

## Résilience et traçabilité

Chaque appel est corrélé à la transaction et à la session. Les entrées normalisées, sorties, versions, décisions finales et éventuelles dérogations humaines sont auditables conformément à la politique de rétention.

En cas d’indisponibilité, chaque type d’opération applique une stratégie déterministe : refus sécurisé, contrôle renforcé ou file de revue. Une réponse tardive ne peut pas modifier une transaction déjà finalisée.

## Critères d’acceptation

1. Les contrats TypeScript compilent en mode strict.
2. Les décisions et niveaux inconnus sont rejetés.
3. Une évaluation incohérente est refusée avant utilisation.
4. La version du modèle figure dans l’audit.
5. Les seuils `29/30`, `59/60`, `84/85`, `0` et `100` sont testés.
6. Jini ne peut exécuter une action sensible sans autorisation backend.
