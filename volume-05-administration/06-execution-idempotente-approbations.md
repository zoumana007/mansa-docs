# Exécution idempotente des approbations

## 1. Objectif

Une approbation valide autorise une opération, mais ne garantit pas à elle seule que cette opération sera exécutée une seule fois. Le composant d’exécution relie la décision administrative à l’opération métier en imposant une clé d’idempotence et un état terminal explicite.

## 2. États d’exécution

- `READY` : exécution enregistrée mais pas encore commencée ;
- `EXECUTING` : consommation engagée ;
- `SUCCEEDED` : opération métier terminée avec une référence exploitable ;
- `FAILED` : opération échouée avec un code permettant le diagnostic et la reprise contrôlée.

Une exécution dans un état terminal ne peut pas être terminée une deuxième fois.

## 3. Conditions de démarrage

Le démarrage est autorisé uniquement lorsque :

1. la demande liée est dans l’état `APPROVED` ;
2. une clé d’idempotence non vide est fournie ;
3. l’exécution est encore dans l’état `READY` ;
4. l’identifiant de l’approbation correspond à la demande consommée ;
5. les contrôles métier, de risque, de limite et de conformité restent valides.

L’approbation ne dispense jamais de refaire les contrôles qui peuvent évoluer entre la décision et l’exécution.

## 4. Idempotence

La clé d’idempotence est unique dans le périmètre du type d’action et de l’approbation. Elle doit être persistée avec une contrainte d’unicité. Une nouvelle tentative avec la même clé renvoie l’état déjà enregistré au lieu de créer une seconde opération.

Le traitement recommandé est le suivant :

1. ouvrir une transaction de base de données ;
2. verrouiller ou réserver l’exécution `READY` ;
3. passer l’état à `EXECUTING` ;
4. enregistrer l’outbox ou l’instruction métier ;
5. valider la transaction ;
6. exécuter l’adaptateur métier de manière idempotente ;
7. enregistrer `SUCCEEDED` ou `FAILED`.

## 5. Résultat métier

Une réussite conserve :

- la référence de l’opération métier ;
- la date de démarrage ;
- la date de fin ;
- la clé d’idempotence ;
- l’identifiant de l’approbation.

Un échec conserve un code stable et non sensible. Les détails techniques confidentiels restent dans les journaux sécurisés et ne sont pas exposés aux clients.

## 6. Reprise

Une exécution `FAILED` ne doit pas être remise silencieusement à `READY`. Une politique de reprise explicite détermine si :

- la même opération peut être rejouée avec la même clé ;
- une intervention manuelle est obligatoire ;
- une nouvelle approbation est nécessaire ;
- l’opération doit être compensée.

Les reprises sont auditables et ne doivent jamais créer un double débit, un double remboursement ou une double modification de configuration.

## 7. Audit et observabilité

Chaque transition produit un événement append-only comprenant au minimum l’identifiant d’approbation, la clé d’idempotence, l’état précédent, l’état suivant, l’horodatage, l’acteur technique et la corrélation métier.

Les métriques suivent les exécutions en attente, en cours, réussies, échouées, anciennes et reprises. Une exécution `EXECUTING` au-delà du délai attendu déclenche une alerte.

## 8. Correspondance avec le code

Le contrat de référence se trouve dans `packages/security/src/approval-execution.ts` du dépôt `zoumana007/mansa-platform`. Il expose la création, le démarrage, la réussite et l’échec d’une exécution. Les tests unitaires associés couvrent le refus d’une demande non approuvée, la consommation unique et les états terminaux.

## 9. Critères d’acceptation

1. Une demande non approuvée ne peut pas être exécutée.
2. Une clé d’idempotence vide est refusée.
3. Une exécution ne peut être démarrée qu’une seule fois.
4. Une réussite exige une référence métier non vide.
5. Un échec exige un code non vide.
6. Un état terminal ne peut plus être modifié.
7. Le stockage persistant impose l’unicité de la clé d’idempotence.
8. Les transitions sont auditées et corrélées à l’opération métier.
