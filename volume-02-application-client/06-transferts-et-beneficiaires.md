# Transferts et bénéficiaires

## 1. Objectif

Ce document définit le parcours fonctionnel des transferts initiés depuis l’application Client et les règles communes aux autres interfaces Mansa. Le contrat technique correspondant est maintenu dans `mansa-platform/packages/contracts/src/transfer.ts` et `transfer-api.ts`.

## 2. Types de transfert

Les types normalisés sont :

- `INTERNAL` : transfert entre portefeuilles Mansa ;
- `BANK` : transfert vers un compte bancaire partenaire ;
- `MOBILE_MONEY` : transfert vers un portefeuille d’opérateur ;
- `MERCHANT` : transfert ou règlement vers un commerçant ;
- `PUBLIC_SERVICE` : paiement dirigé vers un organisme public.

Le type indique la famille métier. Le partenaire, le rail et la route technique sont sélectionnés côté serveur selon le pays, la devise, la destination, les coûts, la disponibilité et les règles de risque.

## 3. Bénéficiaire

Un transfert externe est préparé à partir d’un bénéficiaire enregistré et autorisé. Le bénéficiaire contient uniquement les données nécessaires à l’acheminement et à l’affichage masqué.

Avant utilisation, le backend contrôle notamment :

1. le statut du bénéficiaire ;
2. la propriété ou le périmètre autorisé ;
3. la validité de la destination ;
4. la méthode de vérification appliquée ;
5. les règles de sanctions, fraude et conformité ;
6. les limites liées au bénéficiaire ou au rail.

Une modification sensible de destination peut imposer une nouvelle vérification ou une période de refroidissement configurable.

## 4. Devis de transfert

`POST /v1/transfers/quotes` calcule un devis temporaire avant la création du transfert.

La demande contient :

- utilisateur propriétaire ;
- portefeuille source ;
- bénéficiaire ;
- montant en unités mineures et devise ;
- référence client facultative.

Le devis retourne :

- un identifiant unique ;
- le type de transfert ;
- le montant ;
- les frais de service, partenaire et taxes ;
- le total ;
- une date d’expiration.

Un devis expiré ne peut pas être utilisé. Toute différence de montant, devise, source ou bénéficiaire impose un nouveau devis.

## 5. Création

`POST /v1/transfers` crée un transfert à partir d’un devis valide. Une clé d’idempotence est obligatoire.

Avant acceptation, le backend vérifie au minimum :

- l’identité et le statut de l’acteur ;
- la propriété du portefeuille ;
- le solde disponible ;
- les limites configurées ;
- la validité du devis ;
- le statut du bénéficiaire ;
- les politiques d’autorisation et de séparation des tâches ;
- les contrôles de conformité et de fraude ;
- la disponibilité du partenaire sélectionné.

Deux appels identiques avec la même clé ne doivent jamais produire deux débits. Une clé réutilisée avec une charge utile différente est rejetée.

## 6. Autorisation forte

Selon le montant, le risque, le pays, le type de transfert ou le profil du client, le transfert peut passer à `PENDING_AUTHORIZATION`.

`POST /v1/transfers/:transferId/authorize` accepte les méthodes :

- `PIN` ;
- `BIOMETRIC` ;
- `OTP` ;
- `STRONG_AUTHENTICATION`.

Le backend ne doit jamais faire confiance à une simple indication locale de réussite biométrique. L’application transmet une preuve ou un jeton vérifiable selon l’intégration retenue. Aucun OTP ou secret ne doit être journalisé.

## 7. Cycle de vie

Les statuts sont :

- `CREATED` ;
- `PENDING_AUTHORIZATION` ;
- `PENDING` ;
- `PROCESSING` ;
- `COMPLETED` ;
- `FAILED` ;
- `CANCELLED` ;
- `REVERSED`.

Les statuts finaux ne sont pas modifiés silencieusement. Une correction financière après exécution doit produire une opération de reversement liée et des écritures distinctes dans le grand livre.

Les principaux codes d’échec sont : fonds insuffisants, limite dépassée, bénéficiaire bloqué, revue conformité requise, partenaire indisponible, destination rejetée et erreur technique.

## 8. Consultation

Les routes de consultation sont :

- `GET /v1/transfers` ;
- `GET /v1/transfers/:transferId` ;
- `GET /v1/transfers/:transferId/receipt`.

La liste est paginée et filtrable par utilisateur, portefeuille source, bénéficiaire, type, statut, référence client et période de création.

Un client ne consulte que ses ressources. Les administrateurs doivent disposer d’une permission et d’une portée explicites. Les consultations sensibles sont auditables.

## 9. Annulation

`POST /v1/transfers/:transferId/cancel` exige un motif non vide.

L’annulation est refusée lorsque :

- le transfert est déjà final ;
- le partenaire l’a rendu irréversible ;
- les écritures imposent désormais un reversement ;
- l’acteur ne possède pas le droit requis ;
- une opération concurrente a déjà modifié le statut.

Les appels concurrents restent idempotents au niveau métier et ne créent pas de compensation multiple.

## 10. Reçu

Le reçu contient la référence, le statut, le type, le montant, les frais, les libellés masqués de la source et du bénéficiaire, ainsi que les dates utiles.

Il ne contient jamais de numéro de carte complet, secret partenaire, code OTP, document KYC ou coordonnée complète non nécessaire.

## 11. Sécurité et audit

Chaque devis, création, autorisation, annulation, échec partenaire et reversement doit rester corrélable avec :

- acteur et type d’acteur ;
- ressource ;
- résultat ;
- identifiant de corrélation ;
- clé d’idempotence lorsqu’elle existe ;
- environnement ;
- horodatage serveur ;
- code d’échec non sensible.

Les charges utiles brutes des partenaires ne sont pas stockées dans les journaux applicatifs lorsqu’elles contiennent des données personnelles ou des secrets.

## 12. Critères d’acceptation

- Un devis expiré ou modifié ne peut pas créer un transfert.
- Deux requêtes identiques avec la même clé ne débitent qu’une fois.
- Un transfert soumis à autorisation ne part pas avant validation réussie.
- Un bénéficiaire bloqué ou hors périmètre ne peut pas être utilisé.
- Les listes sont paginées et limitées au périmètre autorisé.
- Une annulation exige un motif et respecte l’irréversibilité du rail.
- Un statut final ne peut pas être réécrit manuellement.
- Le reçu masque les informations sensibles.
- Tous les changements critiques sont auditables.
- Une indisponibilité partenaire produit un résultat corrélable sans exposer de secret.

## 13. Éléments restant à implémenter

Le contrat cible est défini, mais restent à construire progressivement : contrôleurs NestJS, services d’application, persistance, machine d’état, intégration ledger, politiques d’autorisation, routage partenaire, événements, webhooks, tests contractuels et parcours d’interface.