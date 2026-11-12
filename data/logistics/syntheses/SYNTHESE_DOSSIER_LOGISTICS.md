# SYNTHÈSE DOSSIER LOGISTICS — ANALYSE & PISTES TRANSVERSALES

**Master 2 Data Science & Intelligence Artificielle (Groupe 05) | Année 2026-2027**

---

## SYNTHÈSE EXÉCUTIVE

| Aspect | Résultat |
| :--- | :--- |
| **Fichiers analysés (Blocs 1–3)** | 2 : `inventory_month_end` + `purchase_order_lines` |
| **Données** | ~500 relevés de stock mensuels + 650 lignes de commandes d'achat (2014–2025) |
| **Signal inventaire** | Profil **fort volume** (~5 %) : `medical_consumables` / `protective_equipment` en `case`, concentrés sur **FCL-1010** / **FCL-1020** |
| **Signal commandes** | Extrêmes **prix / quantité** (grants, froid, médical) ; urgence (`emergency`) **sans** délai plus court ; IF/LOF ciblent surtout statuts atypiques |
| **Acteurs recommendations** | **FCL-1010** très présent (stocks + 65 PO) ; **BP-30021** (grants / contrats chers, anomalies) ; **BP-30048** (3 PO, aucune anomalie) ; couple **BP-30021 ∩ FCL-1010 = 0** dans les PO |
| **Piste transversale** | Hub stock **FCL-1010** + volumes médicaux / `case` à croiser avec `shipments` / `goods_receipts` / `inventory_adjustments` |

---

## Sommaire du Dossier

1. [Synthèse exécutive](#synthèse-exécutive)
2. [Fiche 1 : Inventaire de fin de mois](#1-fiche-1--inventaire-de-fin-de-mois-inventory_month_endipynb)
3. [Fiche 2 : Lignes de commandes d'achat](#2-fiche-2--lignes-de-commandes-dachat-purchase_order_linesipynb)
4. [Connexion transversale (2 fichiers)](#3-connexion-transversale--les-2-fichiers-faits)
5. [Fichiers encore à croiser](#4-fichiers-encore-à-croiser--auditer)

---

## 1. Fiche 1 : Inventaire de fin de mois (`inventory_month_end.ipynb`)

### 1.1 De quoi parle le fichier ?

Une ligne = **un stock mensuel** pour un couple (`facility_code`, `reporting_month`, `item_category`) : ouverture, réceptions, sorties, usages internes, déchets, clôture, unité et statut.

- **~500** relevés nettoyés × **5 sites** (dont **FCL-1010** : 100 lignes).
- Rôle dans la chaîne : **état des stocks** (aval des commandes / réceptions, amont des expéditions).

### 1.2 Prétraitement et nettoyage

| Problème | Traitement |
| :--- | :--- |
| Doublons stricts (2) + doublons de clé (site, mois, catégorie) | Suppression (1ʳᵉ occurrence) |
| Unité recopiée dans les valeurs de stock | Retrait de l'unité redondante |
| Virgule décimale | `,` → `.` sur flux numériques |
| Casse `inventory_status` | Normalisation minuscules |
| Valeurs manquantes | Aucune |
| Cohérence temporelle clôture → ouverture mois suivant | Vérifiée ≈ **99,6 %** (fiable) |
| Formule de flux ≠ clôture | **Documentée, non « forcée »** (mouvements hors fichier possibles) |
| Encodage | One-Hot `item_category` / `unit` / `inventory_status` → `*_clean_encoded.csv` |

**Biais assumé :** ne pas inventer une correction de flux évite de masquer une incohérence métier utile pour le croisement avec `inventory_adjustments`.

### 1.3 Bloc 1 — EDA

- **Typage** : 6 numériques de flux/stock ; nominales `item_category`, `unit`, `inventory_status`, `facility_code` ; dates → `year` / `month`.
- **Distributions** : stocks/flux très asymétriques (mélange d'unités `case` / `pallet` / `kit`…).
- **Corrélations** : `opening_stock` ↔ `closing_stock` très fortes ; reçus / sorties liés aux niveaux de stock.
- **Crosstabs** : chaque catégorie a souvent une **unité dominante** (ex. agricole ↔ `pallet`, biologie ↔ `sealed_lot`).
- **Irrégularités** : gros volumes médicaux en `case` ; quelques statuts `provisional` / `revised` ; écarts de formule de flux.

### 1.4 Bloc 2 — Clustering

- **Variables** : 6 numériques + One-Hot catégorie / unité / statut. Écartés : `facility_code`, dates.
- **Standardisation** : `StandardScaler` sur les numériques uniquement.
- **Choix de $k$** : silhouette clairement meilleure à **$k=2$ (≈ 0,63)** ; chute dès $k=3$.
- **CAH (Ward)** : accord élevé avec K-Means → groupes **stables**.
- **Typologie** :
  - 🟢 **Cluster 0 (~95 %) — stock standard** : opening/closing ≈ 28 ; unités mélangées.
  - 🔴 **Cluster 1 (~5 %) — fort volume** : opening ≈ 148, closing ≈ 160, reçus ≈ 37 ; surtout `case` ; `medical_consumables` / `protective_equipment` ; sites **FCL-1010** / **FCL-1020**.
- **PCA 2D** : le petit cluster fort volume apparaît nettement à part.

### 1.5 Bloc 3 — Anomalies & patterns

#### A. Stats (IQR / z-score)
- ~**90–100** lignes flaggées (surtout IQR).
- Dominées par `case` + médical / protective / monitoring → **extrêmes de volume**, pas erreurs de saisie.

#### B. Multivarié (Isolation Forest + LOF)
- `contamination=0.05` sur la même matrice que le clustering.
- **IF** : retrouve le profil fort volume / `case` / médical.
- **LOF** : plus dispersé (combinaisons localement atypiques).
- IF ∩ LOF partiel → méthodes **complémentaires**.

#### C. Apriori
- Règles à lift élevé = liens quasi déterministes catégorie ↔ unité (déjà vus en EDA).
- Confirme la structure du fichier, **sans narratif caché** nouveau.

#### D. Croisement qualitatif
- Beaucoup de flags stats/IF recoupent fort stock / medical+case / statut atypique = extrêmes **attendus**.
- Lignes détectées **sans** signal qualitatif (surtout LOF) = pistes cas par cas.

### 1.6 Bilan du fichier

| Profil | Signature | Lecture |
| :--- | :--- | :--- |
| **Standard (~95 %)** | stocks moyens, unités diverses | régime normal |
| **Fort volume (~5 %)** | gros stocks `case` médical / protective sur FCL-1010 / FCL-1020 | hub de rétention / flux lourds |
| **Formule de flux** | ouverture + flux ≠ clôture (souvent) | mouvements non tous tracés ici |

> **Conclusion :** `inventory_month_end` isole un **profil structurel de fort volume** concentré sur quelques sites (dont **FCL-1010**), surtout consommables médicaux en `case`. L'incohérence de flux reste une piste documentaire à croiser avec ajustements / réceptions / expéditions.

---

## 2. Fiche 2 : Lignes de commandes d'achat (`purchase_order_lines.ipynb`)

### 2.1 De quoi parle le fichier ?

Une ligne = **un article commandé** : qui (site `FCL-…`) achète quoi, à qui (partenaire `BP-…`), combien, à quel prix, avec quelle urgence et quel statut.

| Indicateur | Valeur |
| :--- | :--- |
| Lignes brutes / nettoyées | 651 → **650** |
| Période | 2014–2025 |
| Sites / partenaires / articles | 30 / 69 / 27 |
| Focus | FCL-1010 = **65** ; BP-30048 = **3** ; BP-30021 = **11** ; BP-30021 ∩ FCL-1010 = **0** |

Rôle dans la chaîne : **amont** (avant réceptions / stocks / expéditions).

### 2.2 Prétraitement et nettoyage

| Problème | Traitement |
| :--- | :--- |
| 1 doublon de ligne / PO | Suppression |
| `purchase_order_line` toujours = 1 | Colonne non informative (conservée, notée) |
| Dates / numériques en texte, formats mixtes | Conversion `datetime` / float ; dérivation `order_year`, `order_month`, `lead_time_days` |
| Catégories en MAJUSCULES (6) | Même catégories mal normalisées (signal qualité) |
| `declared_cost_center` manquant (3) | Imputation `unknown` |
| Multi-devises par site (13/30) | Conservé (pas forcément une erreur) |
| Encodage | One-Hot item/unit/currency/auth/status/cost_center ; partners/facilities **gardés en texte** (forte cardinalité) |

Exports : `cleaned/purchase_order_lines_clean.csv` (650×19) + `_encoded.csv` (650×63).

### 2.3 Bloc 1 — EDA

- **Numériques** : `ordered_quantity`, `unit_price`, `lead_time_days` — qty/prix très asymétriques (mélange d'unités).
- **Corrélations** : qty ↔ price **négative** (gros volumes = prix unitaire bas ; grants/services = qty 1, prix élevé).
- **Crosstabs** : auth × statut ; catégorie × unité ; top partners × facilities.
- **Temporel** : volumes par année/mois ; focus FCL-1010/1011 + BP-30048/30021.
- **Irrégularité métier** : délai moyen `emergency` ≈ `ordinary` (~23–24 j) → l'urgence **ne raccourcit pas** vraiment le lead time.

### 2.4 Bloc 2 — Clustering

- **Variables** : qty, price, lead_time + One-Hot catégorie / unité / auth / statut. Partners/facilities hors matrice (analysés à part).
- **$k$ retenu** : **3** (meilleure silhouette ≈ **0,23**).
- **Tailles K-Means** : {0: 162, 1: 80, 2: 408}.
- **CAH** : tailles différentes (79 / 309 / 262) → accord partiel, groupes moins nets qu'en inventaire.
- **PCA** : variance ≈ **28 % + 18,5 % = 46,5 %**.
- **Focus recommendations** :
  - FCL-1010 : surtout cluster 2 (42/65)
  - BP-30048 : clusters 2 et 0 (3 lignes)
  - BP-30021 : réparti sur les 3 clusters

### 2.5 Bloc 3 — Anomalies & patterns

#### A. Stats (IQR / z-score) — **113 / 650**
- Surtout **prix** (80 IQR, 13 \|z\|>3) et **quantités** (33) ; **0** anomalie lead_time IQR/z.
- Tops : partenaire **BP-30078** (18) ; site **FCL-1011** (30) ; catégories `cold_storage_equipment`, `research_contract`, `medical_equipment`, `grant`.
- Exemples extrêmes : grants ~**28 k** ; quantités ~**90 cases**.

#### B. Multivarié — **IF 33 / LOF 33 / ∩ 18**
- IF/LOF ciblent souvent des commandes **pending / delayed / cancelled** avec combo atypique (urgence + délai long, service cher…).
- Ex. IF∩LOF : `PO-2022-00076` (emergency + 41 j + delayed).

#### C. Apriori
- 538 itemsets / 513 règles (support 0,03, conf. ≥ 0,5, lift > 1,3).
- Meilleures règles = catégorie ↔ unité / niveaux prix-qty (redondant avec EDA).

#### D. Croisement + cas prioritaires
- Stats ∩ qualitatif (prio BP/FCL ou auth forte) : 54/113 ; IF 15/33 ; LOF 18/33.
- **FCL-1010** : 9 anom. stats + 1 IF (annulée) — froid / grants / gros volumes médicaux.
- **BP-30021** : 3 stats + 2 LOF (grants / contrat / service).
- **BP-30048** : **0** anomalie.
- **BP-30021 ∩ FCL-1010** : **aucune** ligne dans ce fichier.

### 2.6 Bilan du fichier

| Signal | Lecture |
| :--- | :--- |
| Prix / qty extrêmes | Souvent **attendus** (grants, froid, médical, units hétérogènes) |
| IF/LOF + statuts anormaux | Pistes opérationnelles plus « suspectes » |
| Urgence sans délai court | Dysfonctionnement process / déclaration |
| FCL-1010 / BP-30021 | Acteurs à suivre ; BP-30048 calme ici |

> **Conclusion :** `purchase_order_lines` documente l'**amont achat**. Les « anomalies » numériques sont souvent des commandes chères/volumineuses légitimes ; le vrai intérêt narratif est le **comportement des acteurs prioritaires** et les commandes à **statut anormal**, à croiser avec réceptions / stocks / expéditions.

---

## 3. Connexion transversale — les 2 fichiers faits

| Dimension | `inventory_month_end` | `purchase_order_lines` | Lecture croisée |
| :--- | :--- | :--- | :--- |
| **FCL-1010** | Hub fort volume (cluster ~5 %, médical/`case`) | 65 commandes ; 9 anom. stats | Site **central** des recommendations |
| **FCL-1020** | Aussi dans le cluster fort volume | Présent dans tops prix / IF | 2ᵉ hub stock / flux |
| **BP-30021** | (pas dans inventaire — pas de partenaire) | 11 PO, grants/contrats, anomalies | Fournisseur sensible **côté achat** |
| **BP-30048** | — | 3 PO, **aucune** anomalie | Calme dans ce fichier (à vérifier ailleurs) |
| **BP-30021 ∩ FCL-1010** | — | **0** PO | Le lien n'est **pas** direct dans les commandes → chercher via **shipments / receipts / ERP** |
| **Médical / `case`** | Signature du cluster fort volume | Gros volumes commandés en `case` | Cohérence amont↔stock |

### Hypothèse de travail (à confirmer)

1. **FCL-1010** concentre des **stocks lourds** de consommables médicaux → cohérent avec des **gros achats** / flux de transport (déjà évoqué côté `shipments` dans le notebook dédié).
2. **BP-30021** apparaît comme partenaire **à risque côté PO** (montants élevés, flags LOF), mais **pas comme fournisseur direct de FCL-1010** dans `purchase_order_lines` → le lien recommendations passe probablement par un **autre fichier** (expéditions, partenaires ERP, réceptions).
3. L'écart **formule de stock** (inventaire) suggère des mouvements **non tracés** ici → rôle attendu de `inventory_adjustments` / `goods_receipts`.

---

## 4. Fichiers encore à croiser / auditer

| Fichier | Intérêt pour la suite |
| :--- | :--- |
| `shipments` | Destinations FCL-1010, volumes `medical_consumables`, lien BP-30021 |
| `goods_receipts` | Passage commande → stock (fermeture de la boucle PO) |
| `inventory_adjustments` | Expliquer les écarts ouverture+flux ≠ clôture |
| `erp_business_partners` / `erp_facilities` | Statuts partenaires (ex. `review`), géographie, gouvernance FCL/BP |

---

**Dossier logistics (état actuel) :** 2 fichiers Blocs 1–3 terminés — **hub stock FCL-1010** + **achats atypiques BP-30021 / volumes médicaux** établis ; couple BP-30021↔FCL-1010 **absent des PO** → enquête à poursuivre sur expéditions / réceptions / ERP.
