# Moteur de décision d’accès Mansa

## 1. Objet

Ce document spécifie la première tranche exécutable du moteur de décision partagé pour les usages `TOLL`, `PARKING`, `PUBLIC_TRANSPORT`, `CAMPUS`, `EMPLOYEE_ACCESS`, `FUEL_FLEET`, `CANTEEN` et `EVENT`.

Le moteur ne pilote aucun matériel directement. Il reçoit une requête déjà normalisée, un credential éventuel, un droit éventuel et l’état de service du site. Il retourne uniquement une décision structurée `ALLOW`, `DENY` ou `REVIEW` accompagnée d’un motif stable.

L’implémentation de référence est :

`mansa-platform/packages/contracts/src/access-decision-engine.ts`.

## 2. Ordre déterministe des contrôles

L’ordre de décision est volontairement stable afin qu’une même entrée produise toujours le même motif principal :

1. validation minimale de `requestId` et `correlationId` ;
2. cohérence du périmètre organisation/site de l’état de service ;
3. disponibilité du service ;
4. disponibilité du moyen de paiement demandé ;
5. existence du credential ;
6. appartenance du credential à l’organisation ;
7. statut du credential ;
8. existence et cohérence du droit ;
9. statut du droit ;
10. fenêtre temporelle ;
11. lieu autorisé ;
12. produit autorisé ;
13. nombre maximal d’utilisations ;
14. plafond monétaire ;
15. politique de contrôle manuel ;
16. règles de correspondance credential/plaque ;
17. autorisation.

Le moteur ne doit pas varier cet ordre selon l’interface appelante.

## 3. Isolation par organisation

Un credential d’une autre organisation ne doit jamais être reconnu comme credential valide du tenant courant. Dans ce cas, la réponse reste volontairement `CREDENTIAL_UNKNOWN` afin de ne pas révéler l’existence d’un identifiant appartenant à une autre organisation.

Un `AccessServiceAvailability` fourni pour une autre organisation ou un autre site constitue en revanche une incohérence interne du contexte d’évaluation et doit produire une erreur technique avant décision.

## 4. Statut du service

Les statuts `CLOSED` et `DISABLED` produisent `DENY / SERVICE_CLOSED`.

Les statuts `SUSPENDED` et `MAINTENANCE` produisent `DENY / SERVICE_SUSPENDED`.

Lorsque disponibles, les informations suivantes sont conservées dans la décision :

- `alternativeLocationId` ;
- `publicMessageKey` ;
- moyens de paiement de repli lorsqu’ils restent pertinents.

Le statut `DEGRADED` n’interdit pas automatiquement le passage : les capacités encore disponibles doivent être évaluées explicitement.

## 5. Moyen de paiement indisponible

Lorsqu’un moyen de paiement est demandé et qu’il n’appartient pas à `availablePaymentMethods`, la décision est :

```text
DENY
PAYMENT_METHOD_UNAVAILABLE
```

La réponse peut contenir `fallbackPaymentMethods` afin que la borne ou l’application propose uniquement les moyens réellement utilisables.

Cette règle permet notamment à une borne de péage de désactiver l’espèce si le rendu de monnaie est indisponible tout en conservant carte ou QR.

## 6. Credentials

Un credential absent produit `CREDENTIAL_UNKNOWN`.

Tout statut autre que `ACTIVE` produit `CREDENTIAL_INACTIVE`.

La référence physique seule ne suffit jamais. La décision utilise le credential résolu côté serveur et vérifie son organisation et son état courant.

## 7. Droits

Un droit est considéré absent lorsqu’il n’existe pas ou lorsqu’il ne correspond pas simultanément :

- à l’organisation de la requête ;
- au sujet du credential ;
- au cas d’usage demandé.

Un droit existant mais non `ACTIVE` produit `ENTITLEMENT_INACTIVE`.

Une requête avant `validFrom` ou après `validUntil` produit `OUTSIDE_VALIDITY_WINDOW`.

## 8. Restrictions métier

Les restrictions suivantes sont évaluées sans dépendance au matériel :

- `allowedLocationIds` -> `LOCATION_NOT_ALLOWED` ;
- `allowedProductCodes` -> `PRODUCT_NOT_ALLOWED` ;
- `maxUsesPerPeriod` -> `USAGE_LIMIT_REACHED` ;
- `amountLimit` -> `AMOUNT_LIMIT_EXCEEDED`.

Pour un plafond monétaire, une devise différente est refusée au même titre qu’un montant supérieur. Aucun taux de change implicite n’est appliqué par le moteur d’accès.

## 9. Plaque et télépéage

Lorsque la politique est `CREDENTIAL_AND_PLATE_REQUIRED`, une plaque absente produit `PLATE_UNREADABLE`.

Si la plaque attendue est connue dans les métadonnées du credential et que la plaque observée diffère, la décision devient `PLATE_MISMATCH`.

La politique `CREDENTIAL_VALID_PLATE_MISMATCH_DENY` refuse également une divergence explicite.

La caméra ANPR et le lecteur UHF restent des sources d’observation. La décision finale reste côté plateforme.

## 10. Contrôle manuel

La politique `MANUAL_REVIEW` retourne toujours :

```text
REVIEW
MANUAL_REVIEW_REQUIRED
```

Elle ne doit jamais être transformée silencieusement en `ALLOW` par une interface, un lecteur ou un contrôleur de voie.

## 11. Autorisation

Une requête qui franchit tous les contrôles produit :

```text
ALLOW
ENTITLEMENT_VALID
```

La décision contient les identifiants du credential, du sujet et du droit. Si un montant était demandé, celui-ci est retourné dans `approvedAmount` sans transformation.

## 12. Hors périmètre de cette tranche

Cette première fonction pure ne remplace pas :

- la résolution d’un credential à partir d’un lecteur ;
- la lecture réelle ANPR ;
- le calcul tarifaire ;
- le débit financier ;
- la réservation de quota concurrente ;
- l’enregistrement d’usage ;
- les politiques offline signées ;
- l’audit persistant ;
- le pilotage de barrière ;
- la gestion de matériel défaillant par adaptateurs.

Ces étapes doivent entourer le moteur dans des services applicatifs transactionnels et idempotents.

## 13. Tests automatisés

La suite :

`mansa-platform/packages/contracts/test/access-decision-engine.test.mjs`

couvre au minimum :

1. autorisation d’un télépéage valide avec plaque correspondante ;
2. credential inconnu ;
3. credential révoqué ;
4. droit absent ;
5. droit suspendu ;
6. droit expiré ;
7. lieu interdit ;
8. produit interdit ;
9. quota d’usage atteint ;
10. plafond monétaire dépassé ;
11. service fermé ;
12. moyen de paiement indisponible ;
13. plaque illisible ;
14. plaque différente ;
15. revue manuelle ;
16. rejet d’un contexte de service provenant d’un autre tenant.

## 14. Étape suivante

La prochaine tranche doit créer un service applicatif d’accès qui :

- résout credential et entitlement depuis la persistance ;
- charge `AccessServiceAvailability` et `AccessTerminalProfile` ;
- appelle `evaluateAccessDecision` ;
- réserve de façon atomique les quotas lorsque nécessaire ;
- enregistre la décision et l’usage avec corrélation ;
- déclenche le paiement ou l’action physique uniquement après autorisation ;
- ajoute des tests PostgreSQL de concurrence et d’isolation multi-tenant.
