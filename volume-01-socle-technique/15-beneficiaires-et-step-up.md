# Bénéficiaires et authentification renforcée

## Objectif

Mansa évalue le niveau de confiance d’un bénéficiaire avant un transfert afin de réduire la fraude, les erreurs d’adresse et les prises de contrôle de compte. La décision est déterministe, indépendante des interfaces clientes et appliquée côté serveur.

## États d’un bénéficiaire

Un bénéficiaire peut être :

- `NEW` : jamais payé par le client ;
- `KNOWN` : déjà payé avec succès ;
- `TRUSTED` : explicitement approuvé par le client après authentification forte ;
- `BLOCKED` : interdit par le client, la conformité ou le moteur de risque.

L’état `TRUSTED` ne contourne jamais les sanctions, les limites transactionnelles, les blocages de compte ou les décisions de conformité.

## Signaux évalués

Le moteur partagé prend en compte :

- l’état du bénéficiaire ;
- l’âge de la relation en minutes ;
- le montant demandé en unité monétaire mineure ;
- le seuil de montant sensible ;
- le niveau de risque de la session ;
- la présence d’une authentification forte récente ;
- la correspondance entre le nom confirmé et le destinataire résolu.

## Décisions

Le contrat retourne :

- `ALLOW` : transfert autorisé sans étape supplémentaire ;
- `REQUIRE_STEP_UP` : authentification renforcée obligatoire ;
- `REQUIRE_REVIEW` : contrôle complémentaire ou temporisation ;
- `DENY_BLOCKED_BENEFICIARY` : bénéficiaire bloqué ;
- `DENY_NAME_MISMATCH` : identité résolue incohérente avec la confirmation du client.

L’ordre de priorité est stable : blocage, incohérence de nom, risque élevé, nouveau bénéficiaire avec montant sensible, puis autorisation.

## Règles minimales

1. Un bénéficiaire bloqué est toujours refusé.
2. Une incohérence de nom est refusée avant toute autre règle.
3. Une session à risque élevé impose une revue, même pour un bénéficiaire de confiance.
4. Un nouveau bénéficiaire payé au-dessus du seuil sensible impose une authentification renforcée.
5. Une authentification forte récente peut satisfaire l’étape renforcée, mais ne supprime pas les autres contrôles.
6. Un montant négatif ou un seuil invalide est rejeté comme erreur de contrat.

## Authentification renforcée

L’authentification renforcée peut utiliser une biométrie locale, un code secret, une clé matérielle ou une autre méthode approuvée. Le backend reçoit uniquement une preuve vérifiable et horodatée ; il ne reçoit ni ne stocke les données biométriques brutes.

La preuve doit être liée à l’utilisateur, à la session, au bénéficiaire, au montant, à la devise et à un identifiant de transaction afin d’éviter sa réutilisation.

## Traçabilité

Chaque décision conserve au minimum : identifiant de bénéficiaire, état, montant, seuil appliqué, niveau de risque, présence et âge de la preuve forte, décision, motif, version de politique, identifiant de corrélation et horodatage.

## Critères d’acceptation

1. Un bénéficiaire `BLOCKED` est refusé quelle que soit la preuve forte.
2. Une incohérence de nom est refusée.
3. Une session à risque élevé produit `REQUIRE_REVIEW`.
4. Un nouveau bénéficiaire au-dessus du seuil sans preuve forte produit `REQUIRE_STEP_UP`.
5. Le même transfert avec une preuve forte récente est autorisé.
6. Un bénéficiaire connu sous le seuil est autorisé lorsque les autres contrôles sont satisfaits.
7. Les valeurs négatives, non entières ou les identifiants vides sont rejetés.
