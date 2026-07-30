# Volume 4 — Ventes, remboursements et fonctionnement hors ligne

## 1. Vente

Une vente TPE contient au minimum : terminal, opérateur, commerçant, point de vente, montant, devise, moyen de paiement, clé d’idempotence et horodatage. Le backend demeure la source de vérité du statut financier.

Les états fonctionnels sont : créée, en attente de présentation, en autorisation, acceptée, refusée, annulée, expirée, remboursée partiellement et remboursée totalement.

## 2. Pourboire et partage

Le pourboire est une ligne distincte du montant principal et reste désactivable par commerce, secteur ou terminal. Le partage d’addition peut créer plusieurs intentions de paiement liées à un même panier ; la somme des paiements validés ne doit jamais dépasser le solde du panier.

## 3. Annulation et remboursement

- Une annulation vise une opération non finalisée ou encore annulable auprès du partenaire.
- Un remboursement vise une vente finalisée.
- Le remboursement partiel est autorisé uniquement dans la limite du montant net encore remboursable.
- Les permissions dépendent du rôle, du plafond et du délai configurés.
- Une validation responsable ou administrateur peut être exigée.
- Chaque tentative conserve son motif, son initiateur et le résultat partenaire.

Aucune suppression de vente n’est autorisée. Les corrections utilisent des opérations compensatoires.

## 4. Reçus

Le reçu peut être imprimé, envoyé par SMS, e-mail, QR ou affiché dans l’application Mansa. Il contient une référence non sensible, le commerce, le point de vente, la date, le montant, le statut et un masque du moyen de paiement lorsque le partenaire l’autorise.

Le reçu ne contient jamais le PAN complet, le cryptogramme, le PIN, un jeton réutilisable ni un secret technique.

## 5. Mode hors ligne

Le hors-ligne n’est pas un fonctionnement par défaut. Il est activé uniquement pour un partenaire, un commerce, un terminal et un moyen de paiement explicitement autorisés.

Contraintes :

- plafond unitaire et cumulé ;
- durée maximale depuis la dernière synchronisation ;
- nombre maximal d’opérations ;
- contrôle local de configuration signée ;
- chiffrement de la file locale ;
- interdiction lorsque le terminal est suspendu, altéré ou trop ancien ;
- synchronisation ordonnée avec clés d’idempotence ;
- arrêt automatique dès qu’une limite est atteinte.

Une opération hors ligne reste « en attente de confirmation » jusqu’à son acceptation par le backend et le partenaire. L’interface ne doit pas la présenter comme définitivement réglée avant confirmation.

## 6. Rapprochement

Le terminal synchronise les opérations, événements de santé et reçus en attente. Le backend compare les données du terminal, du grand livre et du partenaire. Toute différence crée une anomalie de rapprochement sans modification silencieuse des écritures.

## 7. Gestion des erreurs

Les erreurs sont classées en : utilisateur, configuration, réseau, partenaire, sécurité et système. Le message affiché à l’opérateur reste compréhensible et ne révèle aucune information interne. Un identifiant de diagnostic permet au support de retrouver les détails protégés.

## 8. Critères d’acceptation

- Deux envois de la même vente avec la même clé ne créent qu’une opération financière.
- Un remboursement supérieur au reliquat est refusé.
- Une opération hors ligne dépassement de plafond est bloquée localement.
- Une opération en attente n’est jamais affichée comme définitivement acceptée.
- Les écarts de rapprochement sont signalés et auditables.
