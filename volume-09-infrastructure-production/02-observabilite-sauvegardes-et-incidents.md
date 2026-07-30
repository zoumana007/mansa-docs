# Volume 9 — Observabilité, sauvegardes et incidents

## 1. Objectif

Mansa doit permettre de détecter rapidement une dégradation, comprendre son origine, limiter son impact et restaurer le service sans perdre la traçabilité financière.

## 2. Signaux d’observabilité

### Métriques

- disponibilité et latence par route et partenaire ;
- taux d’erreur par code, canal et version ;
- débit de transactions, succès, échecs, annulations et opérations en attente ;
- profondeur des files, âge du plus ancien message et files d’échec ;
- connexions, saturation et réplication de la base ;
- usage CPU, mémoire, stockage et réseau ;
- taux de rapprochement et écarts financiers ;
- tentatives d’authentification, blocages et anomalies de risque.

### Journaux

Les journaux sont structurés et contiennent un identifiant de corrélation, le service, l’environnement, la version, le type d’acteur et le résultat. Les mots de passe, jetons, codes OTP, numéros de carte complets et documents KYC ne sont jamais journalisés.

### Traces

Les appels traversant API, modules internes, files et adaptateurs externes conservent un contexte de trace. Les traces servent au diagnostic mais ne remplacent ni le journal d’audit ni le grand livre.

## 3. Niveaux d’alerte

- `P1` : intégrité financière, fuite de données, indisponibilité générale ou compromission présumée ;
- `P2` : fonction majeure indisponible, partenaire critique dégradé ou retard important de traitement ;
- `P3` : incident limité avec solution de contournement ;
- `P4` : anomalie sans impact immédiat nécessitant correction planifiée.

Une alerte doit être exploitable : symptôme, seuil, environnement, tableau de bord, runbook et propriétaire.

## 4. Gestion d’incident

1. Détecter et qualifier la gravité.
2. Nommer un responsable d’incident et ouvrir un canal dédié.
3. Protéger les fonds et les données avant de chercher une correction définitive.
4. Désactiver un partenaire, un canal ou une fonction via configuration lorsque nécessaire.
5. Préserver les preuves et noter chaque décision horodatée.
6. Informer les parties concernées selon les obligations contractuelles et réglementaires.
7. Restaurer, vérifier les soldes et rapprocher les opérations.
8. Produire un compte rendu sans recherche de culpabilité, avec actions datées et responsables.

## 5. Sauvegardes

- Sauvegardes automatiques chiffrées de PostgreSQL.
- Conservation selon une politique documentée par environnement.
- Copies isolées du compte ou projet de production.
- Vérification régulière de l’intégrité des sauvegardes.
- Tests de restauration complets sur un environnement isolé.
- Sauvegarde des configurations critiques, métadonnées de stockage et éléments nécessaires à la reconstruction.

Les sauvegardes ne remplacent pas le grand livre, les journaux d’audit ni les exports de rapprochement.

## 6. Continuité et reprise

Avant production, chaque composant reçoit des objectifs RPO et RTO approuvés. Le plan de reprise documente : ordre de restauration, dépendances externes, DNS, clés, base, files, stockage, validation métier et communication.

Les exercices de reprise sont réalisés périodiquement et après tout changement majeur d’architecture.

## 7. Contrôles financiers après incident

- Comparer les écritures internes aux confirmations partenaires.
- Identifier doublons, absences, opérations bloquées et compensations.
- Interdire toute modification directe silencieuse d’un solde.
- Corriger uniquement par écritures explicites, justifiées et approuvées.
- Conserver les rapports et preuves selon la politique de conservation.

## 8. Critères de validation

- Tableaux de bord disponibles pour les parcours critiques.
- Alertes testées et reliées à un runbook.
- Corrélation possible d’une transaction de bout en bout.
- Restauration testée avec mesure réelle du RPO et du RTO.
- Procédure P1 simulée avec fermeture des actions correctives.
