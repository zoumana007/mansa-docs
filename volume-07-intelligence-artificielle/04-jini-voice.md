# Jini Voice — téléphonie intelligente professionnelle

## 1. Positionnement

Jini Voice est un module professionnel indépendant de téléphonie intelligente. Il peut être intégré à Mansa, mais il doit aussi pouvoir être vendu ou loué séparément à des entreprises, travailleurs indépendants, administrations, collectivités et services publics.

Jini Voice n’est pas une application grand public d’appels ou de messagerie entre particuliers. Il répond aux appels professionnels, comprend la demande, fournit une information autorisée, collecte les éléments nécessaires et transmet les résultats au système métier de l’organisation.

## 2. Intégration prioritaire

L’intelligence conversationnelle doit être intégrée en priorité dans l’application Commerçant et dans les outils métier concernés. Le Hub / Annuaire conserve un rôle de découverte, de routage et de mise en relation.

Deux modes sont prévus :

1. **Organisation connectée** : l’appel, l’intention et les données structurées sont transmis directement à l’application Commerçant, au système de caisse, au CRM, à l’agenda ou au service public concerné.
2. **Organisation non équipée** : Jini Voice utilise un espace léger dans le Hub / Annuaire pour remettre les demandes, commandes, rendez-vous et messages au professionnel.

Cette séparation évite de transformer l’Annuaire en plateforme métier complète et limite les coûts d’infrastructure.

## 3. Fonctions couvertes

- répondre aux appels entrants et initier des appels autorisés ;
- donner des informations validées par l’organisation ;
- prendre une commande ;
- modifier ou annuler une commande selon les règles du commerce ;
- créer, modifier ou annuler un rendez-vous ;
- fournir des informations de paiement sans manipuler de secret bancaire ;
- ouvrir une demande de support ;
- transférer l’appel à un humain ;
- produire une donnée métier structurée à partir de la conversation ;
- envoyer un récapitulatif au client et à l’organisation lorsque cela est autorisé.

## 4. Règles de confirmation

Jini Voice ne doit jamais finaliser silencieusement une action ayant un impact financier, contractuel, médical, administratif ou logistique important.

Une confirmation explicite est obligatoire pour :

- le contenu final d’une commande ;
- le montant et le mode de paiement ;
- l’adresse ou le point de livraison ;
- la date et l’heure d’un rendez-vous ;
- une annulation ;
- la transmission de données sensibles ;
- toute action marquée comme sensible par l’organisation.

En cas de faible confiance, d’ambiguïté ou de conflit entre plusieurs informations, le système demande une précision ou transfère à un humain.

## 5. Conservation limitée

Les enregistrements et transcriptions doivent avoir une durée de conservation courte et configurable. Le réglage recommandé par défaut est :

- transcription brute : quelques heures à quelques jours ;
- enregistrement audio : désactivé ou conservé pendant la durée minimale nécessaire ;
- donnée métier structurée : conservée dans le système de l’organisation selon ses propres obligations ;
- journaux techniques : conservés sans contenu conversationnel complet et avec identifiants masqués.

Trois politiques sont prévues :

- `EPHEMERAL` : suppression très rapide après extraction des données utiles ;
- `SHORT_TERM` : conservation courte pour contrôle qualité, litige ou reprise ;
- `ORGANIZATION_MANAGED` : conservation gérée par l’organisation dans son propre stockage, dans les limites légales applicables.

Le produit doit permettre à l’organisation d’exporter ses données avant suppression. L’export est chiffré, temporaire, traçable et destiné au stockage contrôlé par l’organisation. Aucun export ne doit contenir de secret technique, de numéro de carte complet, de code OTP ou de donnée non autorisée.

## 6. Minimisation des coûts

Pour réduire les coûts :

- l’audio brut n’est pas conservé par défaut ;
- la transcription complète est supprimée rapidement ;
- seules les données métier structurées utiles sont conservées ;
- les traitements lourds sont déclenchés uniquement lorsque nécessaire ;
- les organisations déjà équipées reçoivent directement les données dans leur application ;
- le Hub sert de solution légère pour les professionnels sans logiciel métier ;
- les modèles, fournisseurs téléphoniques et durées de conservation sont configurables par pays et formule commerciale.

## 7. Sécurité et conformité

- annoncer clairement l’utilisation d’un assistant automatisé lorsque la réglementation ou la politique de l’organisation l’exige ;
- obtenir les consentements nécessaires avant enregistrement ;
- chiffrer les flux et les données au repos ;
- masquer les numéros et identifiants dans les journaux ;
- appliquer RBAC/ABAC et séparation des tâches ;
- utiliser une clé d’idempotence pour toute action métier ;
- tracer les confirmations, transferts humains, exports et changements de politique ;
- interdire à l’IA de demander ou répéter un code secret, un OTP, un code PIN ou un numéro de carte complet ;
- isoler chaque organisation et chaque environnement ;
- permettre le blocage immédiat d’un numéro, d’une organisation, d’une intention ou d’un fournisseur.

## 8. Contrats techniques associés

Le dépôt `mansa-platform` définit les premiers contrats dans :

- `packages/contracts/src/voice.ts` : appels, participants, intentions, rétention et exports ;
- `packages/contracts/src/voice-api.ts` : routes et contrats de transport.

Les routes initiales couvrent la création et la consultation d’un appel, la lecture et la confirmation des intentions, la configuration de rétention et les exports de données.

## 9. Prochaines étapes d’implémentation

1. exposer les contrats depuis le package `@mansa/contracts` ;
2. ajouter les tests unitaires des validateurs ;
3. créer le module NestJS `voice` dans l’API Gateway ;
4. définir les adaptateurs de téléphonie et de transcription ;
5. connecter les intentions de commande au module commandes ;
6. connecter les rendez-vous au calendrier métier ;
7. ajouter les files de traitement et stratégies de reprise ;
8. créer les écrans Commerçant et Hub ;
9. ajouter la configuration administrateur ;
10. réaliser les tests de sécurité, charge, conservation et suppression.

## 10. Critères d’acceptation initiaux

- un appel possède un identifiant, une organisation, un statut et une corrélation ;
- une intention indique son type, son niveau de confiance et si une confirmation humaine est requise ;
- les scores de confiance sont exprimés entre 0 et 10 000 points de base ;
- les durées de conservation sont des heures entières bornées ;
- une organisation peut demander un export avant expiration ;
- un export possède un statut, une expiration et une référence de stockage chiffrée ;
- aucune donnée sensible interdite n’apparaît dans les contrats ou journaux ;
- une action métier répétée avec la même clé d’idempotence ne crée pas de doublon ;
- la suppression des données conversationnelles respecte la politique configurée ;
- une organisation sans application Commerçant peut recevoir les demandes dans son espace Hub léger.
