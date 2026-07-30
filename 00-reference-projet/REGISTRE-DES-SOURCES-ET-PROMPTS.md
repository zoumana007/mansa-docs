# Registre des sources et des prompts

## Objectif

Préserver la totalité de l’intention produit sans confondre les demandes originales, les reformulations pour IA, les décisions validées et les hypothèses techniques.

## Catégories

### A — Demandes originales du porteur de projet
À conserver au plus près de leur sens, même lorsque la formulation est informelle. Elles constituent la source fonctionnelle principale.

### B — Prompts reformulés pour Codex / IA de développement
Chaque prompt doit préciser : contexte, périmètre, fichiers concernés, contraintes, critères d’acceptation, commandes de validation et interdictions.

### C — Éléments provenant de DeepSeek
Ils doivent être importés depuis l’export de conversation ou les fichiers fournis. Ne jamais prétendre qu’un texte DeepSeek absent a été récupéré.

### D — Décisions consolidées
Décisions dédupliquées et validées au fil du projet : noms d’applications, technologies, règles de gouvernance, fonctionnement des paiements et exigences de design.

### E — Compléments techniques nécessaires
Éléments ajoutés pour rendre le produit construisible : idempotence, grand livre, migrations, audit, tests, observabilité, sécurité, reprise et gestion des secrets.

## Statut de récupération

- Historique ChatGPT disponible : partiellement consolidé dans l’inventaire et les volumes.
- Prompts donnés dans les échanges : à transformer progressivement en prompts atomiques ordonnés.
- Export DeepSeek complet : **À compléter depuis l’export DeepSeek** si son contenu n’est pas présent dans le dépôt ou l’historique accessible.
- Code historique DeepSeek : **À compléter depuis l’export DeepSeek** ; ne pas intégrer automatiquement du code non vérifié au socle.

## Format obligatoire d’un prompt d’implémentation

```md
# PROMPT-XXX — Titre

## Contexte
## Objectif
## Périmètre autorisé
## Fichiers à lire avant modification
## Exigences fonctionnelles
## Exigences UX
## Exigences sécurité et audit
## Modèles et contrats concernés
## Cas limites
## Tests obligatoires
## Commandes de validation
## Livrables attendus
## Interdictions
```

## Règles anti-perte

1. Ne supprimer aucun besoin lors de la déduplication ; déplacer les variantes dans une section d’historique.
2. Marquer clairement les contradictions et retenir la décision la plus récente explicitement validée.
3. Relier chaque prompt à des identifiants d’exigences.
4. Ne jamais placer plusieurs modules indépendants dans un prompt Codex unique.
5. Un prompt ne doit pas demander une refonte globale lorsqu’un changement local suffit.
6. Toute fonctionnalité financière doit citer les tests d’idempotence, permissions, audit et cohérence du grand livre.
7. Toute fonctionnalité administrable doit préciser la propagation, l’invalidation des caches et le rollback.

## Import DeepSeek à réaliser

Lorsqu’un export est disponible :
1. conserver l’export brut hors des dossiers exécutables ;
2. extraire les messages utilisateur et les réponses utiles ;
3. supprimer les doublons exacts sans perdre les variantes ;
4. classer par application, module et date ;
5. créer une table de correspondance vers les exigences consolidées ;
6. marquer les parties contradictoires ou obsolètes ;
7. ne migrer le code vers `mansa-platform` qu’après revue et validation.
