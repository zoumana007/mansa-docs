# Registre transversal des contrats API

## 1. Objet

Le paquet `@mansa/contracts` expose désormais un point d’entrée transversal `@mansa/contracts/api-contracts`. Il regroupe les contrats HTTP versionnés utilisés par les applications, services backend, outils d’administration et tests d’intégration.

Le registre est défini dans `mansa-platform/packages/contracts/src/api-contracts.ts`. Les sous-chemins spécialisés restent disponibles pour limiter les dépendances d’un module à son seul domaine.

## 2. Domaines publiés

| Domaine | Sous-chemin spécialisé | Préfixe principal |
| --- | --- | --- |
| Identité | `@mansa/contracts/identity-api` | `/v1/identity` |
| Portefeuilles | `@mansa/contracts/wallet-api` | `/v1/wallets` |
| Paiements | `@mansa/contracts/payment-api` | `/v1/payments` |
| Transferts | `@mansa/contracts/transfer-api` | `/v1/transfers` |
| Cartes | `@mansa/contracts/card-api` | `/v1/cards` |
| Commerçants | `@mansa/contracts/merchant-api` | `/v1/merchants` |
| Terminaux | `@mansa/contracts/terminal-api` | `/v1/terminals` |
| Services publics | `@mansa/contracts/public-services-api` | `/v1/public-services` |
| Notifications | `@mansa/contracts/notification-api` | `/v1/notifications` |
| Support | `@mansa/contracts/support-api` | `/v1/support` |
| Bénéficiaires | `@mansa/contracts/beneficiary-api` | `/v1/beneficiaries` |
| Intelligence et risque | `@mansa/contracts/intelligence-api` | `/v1/intelligence` |
| Analytics | `@mansa/contracts/analytics-api` | `/v1/analytics` |

## 3. Règles d’utilisation

- Une application peut importer le registre transversal pour la génération de clients, la documentation ou les tests de cohérence.
- Un module métier doit préférer son sous-chemin spécialisé afin de réduire le couplage.
- Les constantes de routes et méthodes constituent la référence technique ; les contrôleurs ne doivent pas recopier des chemins divergents.
- Les types de requête et de réponse ne remplacent pas la validation d’exécution à l’entrée de l’API.
- Les commandes de mutation sensibles conservent une clé d’idempotence, un contexte d’autorisation et les références de corrélation nécessaires.
- Aucun secret, jeton, identifiant partenaire réel ou donnée personnelle ne doit apparaître dans ces contrats.

## 4. Compatibilité

Toute rupture de chemin, méthode, champ obligatoire, statut ou sémantique exige une nouvelle version d’API ou une stratégie de migration explicite. Les ajouts optionnels compatibles peuvent rester dans la version courante lorsqu’ils ne modifient pas le comportement existant.

Le paquet doit être construit avant publication. Le fichier `package.json` expose le registre via `./api-contracts` et chaque domaine via son sous-chemin propre.

## 5. Contrôles de cohérence

Pour chaque évolution :

1. vérifier que le fichier `*-api.ts` existe et compile ;
2. vérifier que son sous-chemin est déclaré dans `packages/contracts/package.json` ;
3. vérifier qu’il est exporté par `api-contracts.ts` ;
4. comparer les routes aux catalogues du dépôt `mansa-docs` ;
5. exécuter le typecheck du paquet contrats ;
6. tester les règles d’idempotence, pagination, erreurs et autorisation applicables.

## 6. Critères de recette

- L’import depuis `@mansa/contracts/api-contracts` résout tous les catalogues listés.
- Chaque sous-chemin spécialisé reste importable indépendamment.
- Aucun doublon de symbole incompatible n’empêche la compilation du registre.
- Les routes documentées correspondent aux constantes du code.
- Une suppression ou rupture non versionnée échoue lors des contrôles de compatibilité.
