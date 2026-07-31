# Transferts wallet à wallet

## Objectif

Le transfert interne déplace un montant entre deux wallets Mansa sans passer par un partenaire externe. Il doit rester traçable, idempotent et atomique.

## Invariants métier

- Le wallet source et le wallet destination doivent exister.
- Les deux wallets doivent être actifs.
- La devise du montant doit être identique à celle des deux wallets.
- Le montant doit être strictement positif.
- Le wallet source doit disposer d’un solde disponible suffisant.
- Un transfert ne doit jamais produire de solde négatif.
- Le débit et le crédit doivent partager la même date métier.
- L’identifiant de transaction généré ne doit pas être vide.
- Une clé d’idempotence déjà liée au même transfert doit rejouer le résultat sans nouvelle mutation.
- Une clé d’idempotence liée à un autre transfert doit être refusée.

## Atomicité attendue

L’exécuteur de domaine prépare le débit, le crédit et le résultat de transaction. L’adaptateur de persistance doit l’exécuter dans une transaction PostgreSQL unique avec verrouillage des lignes wallet concernées.

La même transaction de base de données doit enregistrer :

1. le nouveau solde du wallet source ;
2. le nouveau solde du wallet destination ;
3. le résultat du transfert ;
4. les écritures comptables associées ;
5. les événements outbox nécessaires aux notifications et aux analytics.

Toute erreur avant le commit doit annuler l’ensemble des mutations. Aucun état partiel ne doit être visible.

## Concurrence

Les wallets doivent être verrouillés dans un ordre déterministe, par exemple par identifiant croissant, afin de limiter les interblocages. Le contrôle du solde doit être effectué après acquisition du verrou.

## Erreurs fonctionnelles

- `TransferWalletNotFoundError` : wallet source ou destination absent.
- `TransferCurrencyMismatchError` : devises incompatibles.
- Solde insuffisant : refus sans persistance.
- Wallet suspendu ou fermé : refus sans persistance.
- Conflit d’idempotence : refus sans nouvelle mutation.

L’API Gateway traduit ces erreurs en réponses stables, sans exposer de détails internes.

## Critères d’acceptation

- Un transfert XOF valide débite et crédite exactement le même montant.
- Les deux wallets reçoivent la même date de mise à jour.
- Un wallet absent ne provoque aucune sauvegarde.
- Une incompatibilité de devise ne modifie aucun solde.
- Un solde insuffisant ne modifie aucun solde.
- Un rejeu idempotent ne débite pas une seconde fois.
- Une panne lors de l’enregistrement du second wallet annule aussi le premier enregistrement.
- Les tests de concurrence démontrent qu’un wallet ne peut pas être dépensé deux fois.

## État d’implémentation

Le package `@mansa/domain` contient le service de transfert, le port de dépôt wallet et l’exécuteur wallet. Les tests unitaires couvrent le chemin nominal, les wallets absents, les devises incompatibles et le solde insuffisant.

Le prochain lot doit fournir l’adaptateur Prisma transactionnel, les écritures de grand livre en partie double et les tests d’intégration PostgreSQL.
