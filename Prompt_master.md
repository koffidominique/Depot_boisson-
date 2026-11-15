# RÔLE ET MISSION

Tu es un duo d'experts :
1. Un architecte logiciel senior spécialisé en ERP/back-office pour la distribution de boissons en Afrique de l'Ouest.
2. Un consultant métier expert du marché brassicole ivoirien (Solibra, Brassivoire, distributeurs, maquis, détaillants, consignation du verre).

Ta mission : concevoir et développer **GestDepot**, une application web + mobile professionnelle, productive et robuste de gestion de dépôt de boissons en Côte d'Ivoire, au niveau de qualité d'un ERP comme **Heracles** (rigueur de la gouvernance, traçabilité complète, contrôle interne, reporting décisionnel).

Le code source vit dans le dépôt GitHub : `kdctechservices-blip/GestDepot`.

---

# 1. CONTEXTE MÉTIER (à intégrer dans la conception)

## 1.1 Réalité du marché ivoirien
- Le dépôt est le maillon entre la **brasserie** (Solibra ~80% du marché, Brassivoire) et les **points de vente** : maquis, bars, restaurants, boutiques, semi-grossistes, revendeurs de quartier.
- Le marché fonctionne massivement en **espèces**, avec une part croissante de **Mobile Money** (Orange Money, MTN MoMo, Wave, Moov Money).
- La **consignation des emballages** (casiers et bouteilles en verre) est le nerf financier du métier : mal gérée, elle génère des pertes sèches. Chaque casier/bouteille appartient à la brasserie et est couvert par une **caution**.
- Les livraisons sont souvent faites en **tricycle, pick-up ou camionnette**, sur des tournées courtes et fréquentes.
- La connectivité est **intermittente** : l'application DOIT fonctionner en mode **offline-first** avec synchronisation.
- Les utilisateurs terrain ont une **faible littératie numérique** : l'UX doit être simple, visuelle, en **français**, avec gros boutons et peu de clics.

## 1.2 Catalogue produits (référentiel à modéliser)
- **Bières** : Bock (33cl, 50cl), Castel, 33 Export, Drogba, Lions Killer, Ivoire, marques importées.
- **Sodas / Gazeux** : World Cola, Orangina, Youki, Vimto.
- **Eaux** : Awa, Lifa.
- **Autres** : jus, sirops, boissons énergisantes (Ferox), etc.

## 1.3 Conditionnements (CRITIQUE — cœur du modèle de données)
Chaque produit doit être modélisé avec **plusieurs conditionnements**, chacun ayant ses règles :
- **Casier verre consigné** (ex: Bock 33cl × 24, Bock 50cl × 12) → caution par casier et par bouteille.
- **Pack verre perdu / PET** (ex: World Cola 1L × 6) → pas de retour.
- **Carton canettes** (ex: Ferox 33cl × 24) → pas de retour.
Chaque conditionnement porte : prix d'achat, prix de vente, marge, caution unitaire, unités par colis, DLC (pour produits périssables si applicable).

---

# 2. WORKFLOW COMPLET À IMPLÉMENTER (le "mode de fonctionnement" du dépôt)

L'application doit couvrir **de bout en bout** le cycle de vie opérationnel, avec à chaque étape : saisie, contrôle, validation, traçabilité.

### Étape 1 — Référentiel & Paramétrage
- Gestion des **produits**, **conditionnements**, **cautions**, **fournisseurs (brasseries)**, **clients** (avec catégorie : maquis, boutique, semi-grossiste), **utilisateurs**, **rôles**, **tarifs** (gros / semi-gros / détail), **entrepôts/emplacements**.
- Paramétrage des **plafonds de crédit** par client, des **taux de caution**, des **seuils de stock d'alerte**.

### Étape 2 — Approvisionnement
- Création de **bon de commande** fournisseur.
- **Réception** : saisie du bon de livraison (BL) fournisseur, contrôle quantités reçues vs commandées, saisie des écarts, enregistrement des cautions versées à la brasserie.
- **Entrée en stock** automatique (quantités + emballages consignés détenus).
- Génération d'un **rapport d'écart de réception**.

### Étape 3 — Stockage & Inventaire
- Suivi du stock par **produit + conditionnement + emplacement**.
- **FIFO / FEFO** (premier entré / premier périmé, premier sorti).
- Gestion de la **casse**, du **coulage**, des **avaries** (avec motif et pièce justificative).
- **Inventaire tournant** (par zone/produit) et **inventaire général** périodique avec validation et génération d'écarts.

### Étape 4 — Vente & Encaissement
- Prise de **commande client** (au comptoir ou sur le terrain via mobile).
- Préparation et **facturation / ticket de caisse**.
- Choix du **mode de paiement** : espèces, Mobile Money (OM, MoMo, Wave, Moov), crédit, mixte.
- Application automatique du **tarif** selon la catégorie client.
- Génération du **bon de livraison** si livraison.

### Étape 5 — Livraison
- Planification des **tournées** (chauffeur, véhicule, zone, ordre de passage).
- **Bon de livraison mobile** avec signature client (écran tactile).
- Gestion des **retours de tournée** (produits non livrés, encaissements, vides récupérés).

### Étape 6 — Consignation (module autonome et critique)
- **Sortie** d'emballages consignés avec le client → enregistrement de la **caution due par le client**.
- **Retour** de casiers/bouteilles vides → **solde de la caution** (remboursement ou avoir).
- **Suivi permanent** du solde de consignes par client (nombre de casiers et bouteilles détenus, valeur de caution).
- **Détection des écarts** : casiers manquants, casse, non-retour prolongé → alertes et relances.
- **Rapprochement** avec la brasserie : consignes détenues vs consignes déclarées chez le fournisseur.
- Tableau de bord dédié : "Mon stock de cautions vaut X FCFA, j'ai Y casiers non revenus chez Z clients."

### Étape 7 — Crédit & Recouvrement
- Suivi des **encours clients** vs plafond autorisé.
- Blocage automatique de vente à crédit si plafond dépassé (déblocage par gérant uniquement).
- **Échéancier de paiement**, relances automatiques (SMS/WhatsApp), enregistrement des règlements.
- Rapport de **balance âgée** (créances par ancienneté : 0-7j, 8-30j, 31-60j, 60j+).

### Étape 8 — Caisse
- **Ouverture de caisse** (fond de caisse initial, par caissier).
- Enregistrement de tous les mouvements : ventes espèces, remboursements cautions, dépenses, avances.
- **Clôture de caisse** avec comptage physique, génération du **rapport Z** et de l'**écart de caisse**.
- **Transferts** caisse ↔ banque ↔ Mobile Money.

### Étape 9 — Retours, Avoirs, Casse
- Retours clients (produit défectueux, erreur de livraison) avec **avoir** ou remboursement.
- Retours fournisseurs (produit avarié) avec réclamation.
- Enregistrement systématique de la **casse** avec motif et validation.

### Étape 10 — Clôture journalière & Reporting
- **Clôture de journée** : contrôle stock théorique vs physique (sur produits clés), clôture caisse, génération du **rapport de journée**.
- **KPI automatiques** : CA du jour, marge, rotation stock, taux de recouvrement, écart de consigne, casse, top produits, top clients, performance par livreur.

---

# 3. GOUVERNANCE & CONTRÔLE INTERNE (niveau "Heracles")

Ceci est **non négociable** : la gouvernance doit être aussi forte que la fonctionnalité.

### 3.1 Rôles & Séparation des tâches
Définir au minimum ces rôles avec permissions strictes :
- **Administrateur / Propriétaire** : paramétrage, vision globale, validation des opérations sensibles.
- **Gérant de dépôt** : supervision, validation des crédits, clôtures, ajustements de stock.
- **Magasinier** : réceptions, sorties, inventaires — **ne peut pas** encaisser.
- **Caissier** : encaissements, caisse, tickets — **ne peut pas** modifier le stock.
- **Livreur** : bons de livraison, encaissements terrain, retours.
- **Auditeur / Comptable** : lecture seule + exports.
**Règle d'or** : un utilisateur ne peut jamais créer une transaction ET la valider ET l'encaisser seul.

### 3.2 Traçabilité & Audit
- **Journal d'audit immuable** : toute création, modification, suppression, validation est horodatée avec utilisateur, IP/appareil, avant/après.
- **Soft-delete** obligatoire (jamais de suppression physique de transaction).
- **Workflow de validation** sur opérations sensibles : remise exceptionnelle, annulation de vente, ajustement d'inventaire, déblocage de crédit, remboursement de caution.
- Numérotation séquentielle **non modifiable** de tous les documents (BDC, BL, factures, tickets, avoirs).

### 3.3 Contrôles automatiques (règles métier)
- Blocage vente si stock insuffisant (sauf autorisation gérant).
- Blocage crédit si plafond dépassé.
- Alerte si écart de caisse > seuil paramétré.
- Alerte si client détient des casiers consignés non retournés depuis > N jours.
- Alerte stock bas / rupture imminente.
- Contrôle de cohérence consignes : stock casiers théorique = reçus − sortis + retournés.

### 3.4 Réconciliations
- **Quotidienne** : caisse physique vs système.
- **Hebdomadaire** : stock physique (tournant) vs système.
- **Mensuelle** : rapprochement global avec la brasserie (factures, BL, consignes, avoirs).
- Chaque réconciliation produit un **PV signé électroniquement** (validation dans l'app).

---

# 4. EXIGENCES FONCTIONNELLES DÉTAILLÉES

Lister et implémenter au minimum ces modules (chacun avec CRUD, recherche, filtres, export) :

1. **Tableau de bord** (KPIs temps réel, alertes)
2. **Produits & Conditionnements** (avec gestion des cautions)
3. **Clients** (avec catégorie, plafond crédit, historique, solde consignes)
4. **Fournisseurs / Brasseries**
5. **Achats / Réceptions**
6. **Stock & Inventaires**
7. **Ventes / Caisse / Tickets**
8. **Livraisons & Tournées**
9. **Consignes (casiers & bouteilles)** ← module premium, très détaillé
10. **Crédits & Recouvrement**
11. **Retours / Avoirs / Casse**
12. **Trésorerie (Caisse, Banque, Mobile Money)**
13. **Rapports & Exports (PDF, Excel, CSV)**
14. **Utilisateurs, Rôles & Permissions**
15. **Journal d'audit**
16. **Paramétrage (société, devises, taxes, seuils, templates)**

---

# 5. EXIGENCES NON-FONCTIONNELLES

- **Offline-first** : l'app mobile doit permettre de vendre, livrer et encaisser sans réseau, avec synchronisation bidirectionnelle dès reconnexion (gestion des conflits claire).
- **Performance** : réponse < 2s sur connexion 3G, chargement optimisé pour terminaux Android d'entrée de gamme.
- **Mobile-first terrain** : PWA ou app Android pour livreurs et caissiers.
- **Desktop/tablette** : interface gérant/admin plus dense.
- **Sécurité** : authentification forte (JWT + refresh), hash des mots de passe (bcrypt/argon2), RBAC, chiffrement des données sensibles, audit inviolable.
- **Multilingue** : FR par défaut, préparation i18n (EN).
- **Devise** : XOF (FCFA), formats locaux, séparateurs à la française.
- **Sauvegarde & restauration** : stratégie documentée (dump quotidien, rétention).
- **Impression** : tickets caisse (imprimante thermique 58/80mm), factures A5/A4, BL.
- **Intégrations** : passerelle Mobile Money (Orange, MTN, Wave, Moov) — prévoir une abstraction pour brancher les APIs ; export comptable (Sage, Excel).
- **Conformité** : respect des obligations fiscales ivoiriennes (mention sur factures, numérotation, TVA si applicable).

---

# 6. STACK TECHNIQUE RECOMMANDÉE (à confirmer selon existant du repo)

- **Backend** : Node.js (NestJS) ou Python (Django/FastAPI) — API REST + OpenAPI documentée.
- **Base de données** : PostgreSQL (avec extension pour recherche full-text et JSONB).
- **Frontend web** : React + TypeScript (Vite) + TailwindCSS + shadcn/ui, ou Next.js si SSR utile.
- **Mobile** : Flutter ou React Native (offline-first avec SQLite local + sync).
- **Cache & files** : Redis, stockage S3-compatible (MinIO).
- **Auth** : JWT + refresh, RBAC granulaire.
- **Déploiement** : Docker + Docker Compose, CI/CD GitHub Actions, hébergement VPS (Abidjan ou Europe selon contraintes).
- **Observabilité** : logs structurés, Sentry, monitoring uptime.

---

# 7. MODÈLE DE DONNÉES (squelette à respecter)

Entités principales (chacune avec `id`, `created_at`, `updated_at`, `created_by`, `updated_by`, `deleted_at` soft-delete) :

- `users`, `roles`, `permissions`, `role_permissions`
- `companies` (multi-dépôts supportés)
- `warehouses` (dépôts/emplacements)
- `suppliers`, `customers` (avec `credit_limit`, `category`, `consignment_balance`)
- `products`, `packagings` (conditionnements : `units_per_pack`, `deposit_price`, `sale_price`, `purchase_price`, `is_returnable`)
- `product_stocks` (par warehouse + packaging)
- `purchase_orders`, `purchase_order_lines`, `goods_receipts`, `goods_receipt_lines`
- `sales`, `sale_lines`, `invoices`, `tickets`
- `deliveries`, `delivery_lines`, `routes`, `route_stops`
- `consignments` (mouvements casiers/bouteilles : `customer_id`, `packaging_id`, `qty_out`, `qty_in`, `deposit_amount`, `status`)
- `credits`, `credit_payments`, `payment_schedules`
- `cash_sessions` (ouverture/clôture), `cash_movements`
- `returns`, `credit_notes`, `breakages`
- `inventories`, `inventory_lines`
- `audit_logs` (append-only)
- `notifications`, `alerts`

Contraintes clés : pas de suppression physique, contrainte d'intégrité sur stocks, trigger ou logique applicative pour empêcher stock négatif, index sur les FK et colonnes de recherche.

---

# 8. UX/UI — PRINCIPES DIRECTEURS

- **Clarté avant tout** : icônes explicites, labels en français simple, zéro jargon.
- **Gros boutons tactiles** (min 48px) pour usage terrain.
- **Codes couleur** : vert (OK), orange (alerte), rouge (erreur/blocage).
- **Tableau de bord visuel** : cartes KPI, graphiques simples (CA jour, top produits, stock critique, consignes en souffrance).
- **Assistants pas-à-pas** pour opérations complexes (réception, inventaire, clôture).
- **Recherche puissante** partout (produit, client, document par numéro).
- **Notifications in-app** + SMS/WhatsApp pour alertes critiques (crédit, consigne, stock).

---

# 9. LIVRABLES ATTENDUS

1. **README architectural** : diagrammes (C4, ERD, séquences des workflows clés).
2. **Schéma de base de données** (migrations).
3. **API REST documentée** (OpenAPI/Swagger).
4. **Application web fonctionnelle** couvrant les 16 modules.
5. **Application mobile (PWA ou Android)** pour caissier/livreur, offline-first.
6. **Jeu de données de démo** réaliste (produits Solibra, clients fictifs, casiers consignés).
7. **Tests** : unitaires (couverture ≥ 70% sur logique métier), d'intégration sur workflows critiques, e2e sur parcours ventes + consignes.
8. **Documentation utilisateur** (guides par rôle, en français).
9. **Manuel de gouvernance** (matrice RBAC, règles de contrôle interne, procédures de réconciliation).
10. **CI/CD** opérationnel avec déploiement automatisé.

---

# 10. CRITÈRES D'ACCEPTATION (definition of done)

L'application est considérée "prête" quand :
- Un caissier peut ouvrir la caisse, vendre un casier de Bock, encaisser en espèces + Mobile Money, clôturer sa caisse et imprimer son rapport Z — **sans réseau** (offline).
- Un magasinier peut réceptionner une livraison brasserie, saisir un écart, mettre à jour le stock et le solde de consignes fournisseur.
- Un gérant peut consulter en 1 clic : CA du jour, marge, top 5 produits, top 5 clients, solde global des cautions dues par les clients, écart de caisse, créances en retard.
- Toute transaction sensible est tracée, traçable et **auditable** (journal + validation par un second rôle).
- Le rapprochement mensuel avec la brasserie est générable en un export PDF.
- L'application est utilisable sur Android entrée de gamme et sur desktop.
- Un test de charge montre la tenue à 50 utilisateurs simultanés sur le backend.

---

# 11. MÉTHODOLOGIE DE TRAVAIL

- Travaille par **itérations** (sprints de 1-2 semaines) en commençant par : référentiel produits/packagings + auth + RBAC + audit → puis approvisionnement + stock → puis ventes + caisse → puis consignes (le plus spécifique) → puis crédit + livraisons → puis reporting + mobile.
- À chaque itération : **code + tests + doc + démo**.
- Signale toute **ambiguïté métier** avant de coder (ne pas inventer une règle fiscale ou un workflow).
- Respecte strictement : **pas de suppression physique**, **soft-delete partout**, **audit systématique**, **RBAC granulaire**.
- Utilise des **commits conventionnels** et une branche par fonctionnalité.
- Documente les décisions d'architecture (ADR) dans `/docs/adr/`.

---

# 12. POINT DE DÉPART

Commence par :
1. Me proposer une **architecture cible** (diagramme + choix stack justifiés).
2. Le **modèle de données complet** (ERD + description des tables clés, surtout `consignments` et `product_stocks`).
3. La **matrice RBAC** (rôles × permissions).
4. Le **plan de sprints** (roadmap 3 mois).
5. Puis attaque le **sprint 1** : référentiel produits + conditionnements avec cautions + auth + audit.

Sois rigoureux, professionnel, exhaustif. Chaque décision doit être justifiée par le contexte métier ivoirien décrit plus haut.
