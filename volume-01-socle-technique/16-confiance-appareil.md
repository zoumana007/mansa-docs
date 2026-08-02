# Confiance des appareils et authentification renforcée

## Objectif

Mansa évalue chaque appareil utilisé pour accéder à un compte avant d’autoriser une opération sensible. Cette évaluation complète les contrôles de session, de risque, de bénéficiaire et de limites transactionnelles. Elle est appliquée côté serveur et ne dépend jamais uniquement d’une déclaration de l’application cliente.

## États d’un appareil

Un appareil peut être :

- `NEW` : appareil jamais observé pour le compte ;
- `KNOWN` : appareil déjà utilisé avec succès ;
- `TRUSTED` : appareil explicitement approuvé après authentification renforcée ;
- `COMPROMISED` : appareil signalé perdu, volé, altéré ou associé à une compromission ;
- `BLOCKED` : appareil interdit par l’utilisateur, le support, la conformité ou le moteur de risque.

La confiance n’est jamais permanente. Elle peut expirer, être révoquée ou être invalidée après un changement important de l’appareil.

## Signaux évalués

Le moteur partagé prend en compte au minimum :

- l’identifiant pseudonymisé de l’appareil ;
- son état de confiance ;
- l’âge de la dernière authentification renforcée ;
- l’âge de la dernière utilisation réussie ;
- le niveau d’intégrité remonté par l’attestation de plateforme ;
- la présence d’un débogage, d’un root ou d’un jailbreak détecté ;
- le niveau de risque de la session ;
- le caractère sensible ou non de l’opération.

Aucune donnée biométrique brute, aucun secret matériel et aucun identifiant publicitaire ne doit être conservé dans ce contrat.

## Décisions

Le contrat retourne :

- `ALLOW` : appareil et contexte compatibles avec l’opération ;
- `REQUIRE_STEP_UP` : authentification renforcée récente obligatoire ;
- `REQUIRE_REENROLLMENT` : réenrôlement de l’appareil obligatoire ;
- `REQUIRE_REVIEW` : opération soumise à contrôle supplémentaire ;
- `DENY_BLOCKED_DEVICE` : appareil bloqué ;
- `DENY_COMPROMISED_DEVICE` : appareil compromis ;
- `DENY_INTEGRITY_FAILURE` : attestation ou intégrité insuffisante.

L’ordre de priorité est stable : blocage, compromission, échec d’intégrité, risque élevé, appareil nouveau ou confiance expirée, puis autorisation.

## Règles minimales

1. Un appareil `BLOCKED` est toujours refusé.
2. Un appareil `COMPROMISED` est toujours refusé pour une opération financière.
3. Un échec d’intégrité impose un refus pour une opération sensible.
4. Une session à risque élevé impose une revue, même sur un appareil de confiance.
5. Un appareil nouveau impose une authentification renforcée pour une opération sensible.
6. Une confiance expirée impose un réenrôlement.
7. Les décisions et versions de politique sont journalisées.

## Révocation

La révocation doit être immédiate depuis l’application client, le support et l’administration. Elle invalide les sessions actives liées à l’appareil, empêche l’émission de nouveaux jetons et produit un événement d’audit. Un appareil révoqué ne peut redevenir fiable qu’après un nouvel enrôlement complet.

## Critères d’acceptation

1. Un appareil bloqué produit `DENY_BLOCKED_DEVICE`.
2. Un appareil compromis produit `DENY_COMPROMISED_DEVICE`.
3. Un appareil dont l’intégrité échoue produit `DENY_INTEGRITY_FAILURE` pour une opération sensible.
4. Un appareil nouveau sur une opération sensible produit `REQUIRE_STEP_UP`.
5. Une session à risque élevé produit `REQUIRE_REVIEW`.
6. Une confiance expirée produit `REQUIRE_REENROLLMENT`.
7. Un appareil connu, intègre et à faible risque est autorisé.
8. Les identifiants vides, durées négatives et politiques invalides sont rejetés.
