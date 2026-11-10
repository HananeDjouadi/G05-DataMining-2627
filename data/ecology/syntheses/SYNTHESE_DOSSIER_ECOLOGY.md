# SYNTHÈSE DOSSIER ÉCOLOGIE — ANALYSE & CONNEXION DÉCOUVERTE

**Master 2 Data Science & Intelligence Artificielle (Groupe 05) | Année 2026-2027**

---

## SYNTHÈSE EXÉCUTIVE

| Aspect | Résultat |
| :--- | :--- |
| **Fichiers analysés** | 4 : `bat_surveys` + `environmental_field_reports` + `monitoring_device_operations` + `environmental_monthly` |
| **Données** | 4 992 relevés bats + 30 rapports textuels + ~4 992 cycles capteurs + ~4 992 relevés climatiques |
| **Anomalies centrales** | Chute population −65.9 % (`BAT-0000064`) + preuve matérielle (`EFR-510088`) |
| **Connexion établie** | **Braconnage massif à KNT-KES (avril 2024)** |
| **Contrôles croisés** | Climat innocenté (`environmental_monthly`) ; capteurs = pannes techniques distinctes (`monitoring_device_operations`) |

---

## 📌 Sommaire du Dossier

1. [Synthèse exécutive](#synthèse-exécutive)
2. [BAT_SURVEYS — Relevés de chauves-souris](#1-bat_surveysipynb--4-992-relevés-de-chauves-souris)
3. [ENVIRONMENTAL_FIELD_REPORTS — Rapports textuels](#2️-environmental_field_reportsipynb--30-rapports-textuels)
4. [Connexion découverte](#connexion-découverte--causalité-établie)
5. [Justification des choix (bat + field reports)](#justification-des-choix)
6. [Fiche 3 : Opérations de capteurs (`monitoring_device_operations`)](#4-fiche-3--analyse-des-opérations-de-capteurs-monitoring_device_operationsipynb)
7. [Fiche 4 : Contrôle environnemental mensuel (`environmental_monthly`)](#5-fiche-4--contrôle-environnemental-mensuel-environmental_monthlyipynb)
8. [Synthèse transversale](#6-synthèse-transversale--résolution-de-lenquête-du-dossier)

---

## 1 BAT_SURVEYS.IPYNB — 4 992 Relevés de Chauves-Souris

### Choix Méthodologiques
* **Standardisation** : StandardScaler sur 8 features (nécessaire pour K-Means équitable)
* **Clustering** : k=3 optimal (Silhouette 0.1952, coude visible, dendrogramme CAH confirme)
* **Anomalies** : 3-tier car univariée aveugle aux crashes; **contextuelle révèle BAT-0000064**

### Résultats Clés
| Métrique | Valeur |
| :--- | :--- |
| **Clusters** | 3 groupes d'activité (mégas-colonies / perturbées / basse activité) |
| **Anomalies univariées** | 71 détectées sur colony_estimate |
| **Anomalies multivariées** | 10 consensus (IF + LOF) |
| **ANOMALIE CONTEXTUELLE** | **BAT-0000064 (KNT-KES, 05/04/2024)** : **-65.9% vs avril 2023** |

#### Découverte Clé : BAT-0000064
* **Invisible en analyse globale** (Z-score IF/LOF = normal à l'échelle mondiale)
* **Évident en contexte local** (Z-score KNT-KES = anomalie majeure)
* **Cause non méteo** : weather_interference = 29.9 (normal), donc exclut orage/tempête
* **Conclusion** : Effondrement structurel de la colonie sans explication climatique

---

## 2️ ENVIRONMENTAL_FIELD_REPORTS.IPYNB — 30 Rapports Textuels

### Choix Méthodologiques
* **Vectorisation** : TF-IDF (30 features, min_df=2, max_df=0.8, stop_words) adapté au texte
* **Clustering** : k=2 optimal (Silhouette 0.3841)
* **Anomalies** : Univariée + Multivariée (IF+LOF consensus)

### Résultats Clés
| Métrique | Valeur |
| :--- | :--- |
| **Clusters** | 2 groupes (23 routine / 7 anomalies textuelles) |
| **Anomalies détectées** | 6 rapports flaggés |
| **ANOMALIE TEXTUELLE** | **EFR-510088 (KNT-KES, avril 2024)** : Filets coupés + traces intrusion |

#### Découverte Clé : EFR-510088
**EVIDENCE** (extrait du rapport) :
```
Visual inspection identified fresh handling marks near the secondary entrance,
two short lengths of CUT RESTRAINT MATERIAL,
and disturbed sediment along a route not used by the monitoring team.
```
**Traduction** : Filets cisaillés + traces d'intrusion nocturne à KNT-KES.

---

## CONNEXION DÉCOUVERTE : CAUSALITÉ ÉTABLIE

### Convergence Directe

| Dimension | Preuve Quantitative (BAT-0000064) | Preuve Qualitative (EFR-510088) | Verdict |
| :--- | :--- | :--- | :--- |
| **Site** | KNT-KES (grotte de Kestrel) | KNT-KES | MÊME LIEU |
| **Date** | 05/04/2024 | Avril 2024 | MÊME PÉRIODE |
| **Cause** | Effondrement -65.9% population *Eonycteris spelaea* | Filets coupés + traces intrusion | EFFET ↔ CAUSE |
| **Excluants** | Météo normale (29.9) | Preuve matérielle | BRACONNAGE CONFIRMÉ |

### Conclusion
**`BAT-0000064` est la signature quantitative directe d'un braconnage clandestin massif documenté qualitativement par `EFR-510088`.**

---

## Justification des Choix

| Choix | Raison | Résultat |
| :--- | :--- | :--- |
| **3-tier anomaly detection (univariée/multivariée/contextuelle)** | Univariée seule rate les crashes locaux | BAT-0000064 détecté UNIQUEMENT en contextuel |
| **TF-IDF sur texte** | Capture sémantique des rapports | EFR-510088 flaggée comme anomalie thématique |
| **Z-score contextuel par site** | Utilise historique local comme baseline | Révèle -65.9% comme cassure structurelle |
| **Consensus IF+LOF** | Réduit faux positifs | 6 vrais positifs en environmental_reports |

---

## 4. Fiche 3 : Analyse des opérations de capteurs (`monitoring_device_operations.ipynb`)

### 4.1 Prétraitement et Nettoyage des données

* **Volumétrie initiale** : 4 994 cycles d'opération × 11 colonnes ; une ligne = un appareil sur un site (`site_code` / `device_reference`) entre `operation_start` et `operation_end`.
* **Justifications de nos choix méthodologiques** :
  1. **Doublons** : 2 doublons stricts + doublons d'identifiant `device_operation_id` supprimés $\implies$ **4 992** lignes.
  2. **Correspondance site ↔ appareil** : relation 1-pour-1 (`site_code` ↔ `device_reference`) ; formats `DOP-…` / `DEV-…` homogènes.
  3. **Dates** : `operation_end` au format mixte (`31-May-2025`, `2022/11/30`…) converti en `datetime` ; dérivation de `year`, `month`, `duration_days`.
  4. **Numériques** : `uptime_pct` (virgule décimale) nettoyé ; pourcentages dans [0, 100], aucune valeur impossible.
  5. **Catégorielles** : `maintenance_type` / `record_status` normalisés (casse) ; `record_status` = une seule modalité `closed` → peu informative.
  6. **Manquants porteurs de sens** : `fault_code` manquant à ~96 %. Croisement : rempli à 100 % pour `corrective` / `sensor replacement`, jamais pour `routine` → imputation par **`NO_FAULT`**.
  7. **Prestataire** : 30 trous imputés par le mode historique **du site** (1 prestataire par site sur le reste du jeu).
  8. **Outliers** : uptime / qualité bas **conservés** (expliqués par le type de maintenance).
  9. **Encodage** : One-Hot de `maintenance_type`, `fault_code`, `maintenance_provider` dans `df_encoded` ; `record_status` écartée ; export `cleaned/..._clean.csv` + `..._clean_encoded.csv`.

---

### 4.2 Bloc 1 : EDA et Statistiques Descriptives

* **Typage** : numériques continues (`uptime_pct`, `acoustic_quality`) ; dates → `year` / `month` / `duration_days` ; nominales non ordonnées (`maintenance_type`, `fault_code`, `maintenance_provider`).
* **Distributions** : `describe()`, histogrammes, boxplots ; skew négatif sur uptime / qualité (majorité « bonne », traîne de valeurs basses) — plutôt loi de puissance inversée qu'une normale classique. `duration_days` très concentré (~28–31 j, cycle mensuel).
* **Catégorielles** : barplots `maintenance_type`, `fault_code`, `maintenance_provider` ; interventions non-routine et vrais codes de panne **rares** (~4 %).
* **Corrélations** : Pearson + Spearman + heatmaps + pairplot ; lien positif attendu `uptime_pct` ↔ `acoustic_quality` ; `duration_days` ne corrèle avec rien. Rappel : **corrélation ≠ causalité** (variable cachée = type de maintenance).
* **Tableaux croisés** : `maintenance_type` × `fault_code` / × `provider` ; routine ≈ toujours `NO_FAULT` ; corrective / remplacement concentrent les pannes.
* **Tendances temporelles** : effectifs et moyennes d'uptime / qualité par année et par mois calendaire ; lecture des pics / creux (série 2025 éventuellement tronquée).
* **Irrégularités visuelles** : extrêmes bas d'uptime/qualité = interventions lourdes ; modalités rares = codes de panne hors `NO_FAULT`.

---

### 4.3 Bloc 2 : Clustering et Réduction de Dimensionnalité

* **Variables retenues** : `uptime_pct`, `acoustic_quality` + One-Hot maintenance / panne / prestataire. Écartés : identifiants, dates, `year` / `month`, `duration_days` (quasi constante).
* **Standardisation** : `StandardScaler` sur les 2 numériques uniquement (dummies laissées en 0/1).
* **Choix de $k$** :
  * Coude : forte baisse d'inertie de $k=2$ à $k=3$, puis ralentissement.
  * Silhouette : clairement meilleure à **$k=2$ (≈ 0.65)** ; chute nette dès $k=3$ (≈ 0.24).
  * **Retenu : $k=2$** (meilleure séparation).
* **CAH (Ward) + dendrogramme** : grand saut avant la dernière fusion ; coupe à $k=2$ ; tailles ≈ 190 / 4800. Tableau croisé K-Means ↔ CAH : **accord ≈ 99,7 %** (13 désaccords seulement) → groupes **stables**.
* **Typologie des 2 profils** :
  * 🟢 **Cluster 0 (~4 816, ~96%) — Fonctionnement normal** : uptime ≈ 93.7 %, qualité ≈ 79.6 ; quasi uniquement routine ; `NO_FAULT` ≈ 99.7 %.
  * 🔴 **Cluster 1 (~176, ~4%) — Panne / maintenance lourde** : uptime ≈ 60.7 %, qualité ≈ 61.8 ; 0 % `NO_FAULT` ; ≈ 70 % corrective + ≈ 30 % `sensor replacement` ; pannes typiques `MICROPHONE_DRIFT`, `STORAGE_LIMIT`, `POWER_INTERRUPTION`.
* **PCA 2D** : variance expliquée ≈ **47.5 % + 24 % = 71.5 %** ; PC1 tiré surtout par uptime / qualité acoustique ; le petit cluster apparaît nettement à part.

---

### 4.4 Bloc 3 : Détection d'Anomalies, Patterns & Enquête

#### A. Détection Univariée (IQR & Z-Score)
* Anomalies sur `uptime_pct` / `acoustic_quality` : concentrent les valeurs très basses.
* Recoupent surtout `corrective maintenance` / `sensor replacement` et les vrais `fault_code` → **extrêmes réels**, pas erreurs de saisie (aligné avec le cluster 1).

#### B. Détection Multivariée (Isolation Forest & LOF)
* `contamination=0.02` (~100 anomalies chacune) sur `X_scaled` (même matrice que le clustering).
* **Isolation Forest** : retrouve le profil panne / maintenance lourde ; recoupe très fortement les anomalies stats (≈ 99 / 100).
* **LOF** : autre chose — surtout `routine inspection` + `NO_FAULT` (combinaisons localement atypiques). Peu de recoupement avec l'étape A.
* IF ∩ LOF ≈ **1 ligne** $\implies$ méthodes **complémentaires**.

#### C. Extraction de Règles d'Association (`mlxtend` / Apriori)
* Discrétisation en terciles ; items = région + type maintenance + code panne + uptime/qualité. Prestataire exclu (risque de règles triviales).
* `min_support=0.01` (les pannes sont rares ; un support 0.03 les masquerait), confiance ≥ 0.5, lift > 1.3.
* Règles à lift très élevé : ex. `sensor replacement` → `MICROPHONE_DRIFT` ; `STORAGE_LIMIT` → `corrective maintenance` ; souvent associées à uptime/qualité bas.
* **Bilan** : confirme le lien panne ↔ type d'intervention déjà vu en EDA / clustering, sans nouveau narratif caché.

#### D. Croisement anomalies stats / ML × qualitatives
* Signaux qualitatifs : `fault_code` ≠ `NO_FAULT`, maintenance non-routine, uptime < 70 %.
* IF / anomalies stats recoupent massivement ce qualitatif = extrêmes **documentés**.
* Flags **LOF sans signal qualitatif** = pistes secondaires (cycles routiniers un peu hors norme locale), à inspecter au cas par cas.

---

### 4.5 Bilan du fichier

| Profil | Signature | Lecture |
| :--- | :--- | :--- |
| **Normal (~96 %)** | uptime / qualité hauts, routine, `NO_FAULT` | régime standard des capteurs |
| **Panne (~4 %)** | uptime / qualité bas, corrective / remplacement, vrai `fault_code` | dysfonctionnement technique isolé par clustering + IF + Apriori |
| **LOF « silencieux »** | souvent routine + `NO_FAULT` | atypie locale sans panne déclarée |

> **Conclusion (rôle de ce fichier) :**  
> `monitoring_device_operations` documente la **fiabilité des appareils**. Le signal dominant est technique : un petit groupe d'opérations en **panne / maintenance lourde**, stable entre K-Means, CAH, Isolation Forest et les règles d'association.  
> Ce n'est pas une intrigue « cachée » spectaculaire : c'est la structure naturelle du jeu (≈ 4 % d'interventions non-routine). Utile en transversal pour vérifier si une baisse de mesures sur un site/mois vient d'un **capteur en panne** plutôt que d'un phénomène écologique.

---

## 5. Fiche 4 : Contrôle environnemental mensuel (`environmental_monthly.ipynb`)

### 5.1 Prétraitement et Nettoyage des données

* **Volumétrie initiale** : 4 994 relevés mensuels × 11 colonnes (janvier 2019 → juin 2025), une ligne = un couple (`site_code`, `reporting_month`) sur 64 sites et 3 régions (AVL, KNT, TMB).
* **Justifications de nos choix méthodologiques** :
  1. **Doublons** : 2 doublons stricts + doublons de clé métier (`site_code` + `reporting_month`) supprimés $\implies$ taille finale de **4 992 relevés**.
  2. **Formats incohérents** : dates en texte converties en `datetime` ; séparateurs décimaux `,` → `.` ; unités collées (`25.8 C`, `102.7 mm`) retirées ; `land_use_category` normalisée (18 écritures → 8 catégories réelles).
  3. **Valeurs manquantes « porteuses de sens »** : `weather_exception_code` manquant à ~96 %. Preuve empirique : `DRY_SPELL` ≈ 24 mm de pluie, `HEAVY_RAIN` ≈ 255 mm, absence de code ≈ 160 mm. Imputation par la modalité **`NORMAL`** (l'absence d'information *est* l'information).
  4. **Trous isolés** : `vegetation_cover_pct` (10 NaN) et `pesticide_index` (15 NaN) interpolés **par site** le long de la série temporelle (pas de moyenne globale).
  5. **Outliers** : aucune valeur physiquement impossible (températures 21–32 °C, pourcentages dans [0, 100]). Les extrêmes (mois secs/pluvieux, sites dégradés) sont **conservés** comme réalités environnementales, souvent déjà documentées via `weather_exception_code`.
  6. **Encodage** : One-Hot de `land_use_category` et `weather_exception_code` exporté dans un fichier séparé `_encoded.csv` (le fichier « clean » reste lisible sans dummies).

---

### 5.2 Bloc 1 : EDA et Statistiques Descriptives

* **Saisonnalité / mousson** :
  * Mois les plus pluvieux (août–octobre) = les plus frais ; mois secs (février–avril) = les plus chauds.
  * Les trois régions n'ont **pas le même cycle** : pic de pluie en juillet (KNT), septembre (AVL), **janvier** (TMB, quasi en opposition de phase). Toute comparaison avec `bat_surveys` doit se faire **par région**.
* **Corrélations clés** :
  * Pluie ↔ humidité ≈ **0.75** (relation dominante, quasi linéaire ; Pearson ≈ Spearman).
  * Température ↔ pluie / humidité ≈ −0.22 / −0.27 (effet saisonnier).
  * Couverture végétale ↔ perturbation de l'habitat ≈ −0.31.
  * `pesticide_index` et `data_completeness_pct` peu corrélés au reste (cette dernière = métadonnée de qualité, pas une mesure environnementale).
* **Profils d'usage du sol** :
  * `mixed_agriculture` : moins de végétation, plus de perturbation et de pesticides.
  * `degraded_land` : végétation basse, perturbation élevée.
  * `mixed_forest` : profil inverse (peu perturbé, peu de pesticides).
  * `land_use_category` est **redondante avec `site_code`** (un site = un seul usage) : vigilance avant de les combiner dans un modèle.
* **Dérive temporelle (2019–2025)** :
  * Pas de dégradation de fond 2019–2024 (végétation / perturbation / pesticides stables).
  * Hausse de température ≈ **+1 °C** en 2025 dans les 3 régions (série 2025 tronquée en juin → à confirmer).
  * Baisse de végétation AVL 2025 : 59 % → 55 % (piste à recouper avec les autres fichiers).

---

### 5.3 Bloc 2 : Clustering et Réduction de Dimensionnalité

* **Variables retenues** : 6 mesures numériques (`température`, `pluie`, `humidité`, `végétation`, `perturbation`, `pesticides`) + One-Hot usage du sol / code météo. Écartés : `data_completeness_pct`, identifiants, `year` / `month`.
* **Standardisation (`StandardScaler`)** : nécessaire (échelles hétérogènes : °C, mm, indices 0–100).
* **Choix du nombre de clusters ($k=3$)** :
  * Coude : baisse régulière de l'inertie, **pas de cassure nette**.
  * Silhouette : meilleur score à **$k=2$ (0.199)**, puis **$k=3$ (0.186)** ; chute nette dès $k=4$.
  * Compromis retenu : **$k=3$** (silhouette proche de $k=2$, interprétation plus riche). Scores tous $< 0.25$ $\implies$ groupes **peu séparés** (nuage continu).
  * Dendrogramme hiérarchique (Ward) : 3 grandes branches, recouvrement élevé avec K-Means (~85–90 %).
* **Typologie des 3 profils identifiés** :
  * 🟡 **Cluster 0 (2 036 lignes, ~41%) — Mois secs et chauds** : $T \approx 27.1$ °C, pluie ≈ 99 mm, humidité ≈ 70 %, perturbation faible.
  * 🔵 **Cluster 1 (2 014 lignes, ~40%) — Mois humides et végétalisés** : pluie ≈ 224 mm, humidité ≈ 82 %, végétation ≈ 70 %, perturbation faible.
  * 🔴 **Cluster 2 (942 lignes, ~19%) — Sites perturbés** : perturbation ≈ 50 (×2), pesticides ≈ 31, végétation ≈ 43 % ; dominé par `mixed_agriculture` et `degraded_land` ; quasi absent de TMB.
* **Validation spatiale (PCA 2D)** :
  * Variance expliquée : **29 % (PC1) + 20 % (PC2) = 49 %**.
  * PC1 = axe humide / sec ; PC2 = axe perturbation. Cluster 2 bien séparé en haut ; clusters 0 et 1 se touchent (frontière floue).

---

### 5.4 Bloc 3 : Détection d'Anomalies, Patterns & Enquête

#### A. Détection Univariée (IQR, Z-Score & Z-Score modifié)
* **85 lignes** avec au moins une anomalie IQR et/ou $|z| > 3$ (surtout région **KNT** : 55 lignes).
* Variables les plus touchées : `vegetation_cover_pct`, `habitat_disturbance_index`, `data_completeness_pct`.
* Z-score modifié (médiane / EAM, seuil $|z| > 3.5$) ajouté comme 3ᵉ méthode robuste du cours.
* **Lecture** : ce sont des extrêmes de sites dégradés / mois atypiques, **pas des erreurs de saisie** (cohérent avec la décision de conservation en nettoyage).

#### B. Détection Multivariée (Isolation Forest & LOF)
* Paramétrage : `contamination=0.02` (~100 anomalies chacune), mêmes 6 mesures standardisées que le clustering.
* **Isolation Forest (vue globale)** : retrouve le profil « sites perturbés » — surtout `KNT-GGN` (49 lignes) et `AVL-GRB` (24), usages `degraded_land` / `mixed_agriculture`. Recoupe **32 / 85** anomalies statistiques.
* **LOF (vue locale)** : autre chose — lignes dispersées (TMB 28), habitats peu perturbés (`mixed_forest`, `karst_habitat`, `coastal_habitat`). Seules **6** lignes recoupent l'étape A. Cas le plus suspect : **`TMB-HQL` janvier 2024** (LOF ≈ 1.65).
* Intersection IF ∩ LOF = **15 lignes** seulement $\implies$ méthodes **complémentaires**.
* **KNT-KES avril 2024** : non détecté (LOF ≈ 1.01) — environnement normal ce mois-là.

#### C. Extraction de Règles d'Association (`mlxtend` / Apriori)
* Discrétisation en terciles (`bas` / `moyen` / `haut`) ; items = région + code météo + 6 mesures. `land_use_category` exclue (redondance site ↔ usage).
* **1 078 itemsets**, **219 règles** (confiance ≥ 0.5, lift > 1.3) ; lift max ≈ 2.5 (liens réels mais modérés).
* Meilleures règles = **climatiques** (pluie haute ⇔ humidité haute), déjà vues en EDA ; aucune règle sur `DRY_SPELL` / `HEAVY_RAIN` (trop rares pour le support 0.03).
* **Bilan** : confirme les corrélations connues, **n'apporte pas de narratif caché** nouveau.

#### D. Croisement anomalies stats / ML × qualitatives (nettoyage)
* Signaux qualitatifs mobilisés : code météo exceptionnel, complétude basse ($<$ 95 %), usage `degraded_land`.
* Une partie des flags IF / stats recoupe ces signaux déjà documentés (extrêmes « attendus »).
* Les lignes **détectées sans signal qualitatif** (notamment certains cas LOF) restent les pistes les plus intéressantes pour un narratif propre à ce fichier (ex. `TMB-HQL`).

---

### 5.5 Contrôle croisé KNT-KES & Bilan

| Indicateur | Attendu si cause environnementale | Observation `environmental_monthly` (KNT-KES, avril 2024) |
| :--- | :--- | :--- |
| **Code météo** | `DRY_SPELL` / `HEAVY_RAIN` | **`NORMAL`** (aucun code exceptionnel en KNT mars–mai 2024) |
| **Z-score vs historique du site** | écarts extrêmes (|z| ≫ 2) | Temp. ≈ +2.0 σ ; perturbation ≈ **−2.05 σ** (moins perturbé) ; pluie / humidité / végétation / pesticides \|z\| < 0.6 |
| **Z-score vs autres sites le même mois** | site isolé | Tous \|z\| < 1.5 |
| **Isolation Forest / LOF** | flaggé | **Non détecté** (LOF ≈ 1.01) |
| **Complétude des données** | — | ≈ 92 % (légère baisse, piste qualité de collecte) |

> **Conclusion Forensic (rôle de ce fichier) :**  
> `environmental_monthly` **écarte la cause climatique / environnementale** pour l'effondrement `BAT-0000064` (KNT-KES, avril 2024). Le climat du mois est ordinaire ; la cause est ailleurs — cohérent avec le lien `bat_surveys` ↔ `EFR-510088` (filets coupés / intrusion).  
> En propre, ce fichier documente surtout des **profils** (sites systématiquement perturbés : `KNT-GGN`, `AVL-GRB`) et quelques combinaisons locales atypiques (ex. `TMB-HQL` janv. 2024), plutôt qu'une anomalie narrative unique aussi nette que le raid de Kestrel.

---

### 5.6 Bloc 4 : Graphes (similarité environnementale entre sites)

* **Définition** : graphe **non orienté pondéré** — nœuds = 64 `site_code` ; arête si similarité cosinus des profils moyens ≥ 0.6 ; poids = similarité.
* **Descriptifs** : n = 64, m = 136, densité ≈ 6.7 %, degré moyen ≈ 4.3 ; **pas** de loi de puissance (γ ≈ 0.4).
* **Centralités** : `KNT-KES` connecté (degré 6) mais betweenness faible (rang ~28) ; `KNT-GGN` = pont fort (betweenness #2).
* **Communautés** : Louvain Q ≈ 0.58 (meilleur) ; **KNT-KES et KNT-GGN jamais dans la même communauté**.
* **Process mining** : **non applicable** (fichier ≠ event log).

### 5.7 Bloc 5 : Spatial & Temporel

* **Spatial** : jointure `monitoring_sites` → **64/64** coordonnées (géographie fictive) ; carte ; Moran / LISA / Voronoi **pertinents** (≠ cas à 3 GPS). Moran global peu informatif (3 régions disjointes) ; Moran par région non significatif.
* **Temporel** : 78 mois (2019-01 → 2025-06) ; STL + ACF sur la pluie (saisonnalité ~12 mois) ; anomalies z-score en fenêtre glissante.
* **KNT-KES avril 2024** : **aucun** |z_roll| > 3 sur les 6 mesures ; résidu STL pluie faible → mois **ordinaire**.
* **DTW** : non appliqué (séries déjà alignées) ; K-Means sur profils de pluie recrée les 3 régions.

### 5.8 Bloc 6 : Texte

* **Non applicable** sur ce fichier (pas de champ narratif `text` / `title` / mémo). Documenté dans le notebook pour le rapport. Le corpus textuel ecology reste `environmental_field_reports`.

### 5.9 Récapitulatif type « fiches bat / field reports »

| Métrique | Valeur |
| :--- | :--- |
| **Données** | 4 992 relevés × 64 sites (2019–2025) |
| **Clusters** | 3 profils (secs / humides / perturbés) |
| **Signal propre** | Sites perturbés récurrents (`KNT-GGN`, `AVL-GRB`) |
| **CONTRÔLE CLÉ** | **KNT-KES avril 2024 = environnement NORMAL** |
| **Verdict** | Cause climatique / habitat **écartée** pour `BAT-0000064` |

#### Découverte clé : innocence environnementale de KNT-KES (avril 2024)
* Code météo **`NORMAL`** ; pas de flag IF/LOF ; z-scores historiques modérés ; graphe = nœud banal.
* **Traduction** : le crash bats n'a **pas** de signature climatique dans ce fichier → conforte `EFR-510088` (braconnage).

---

## 6. Synthèse Transversale & Résolution de l'Enquête du Dossier

* **Épicentre confirmé** : `BAT-0000064` @ `KNT-KES` (05/04/2024) — chute de colonie −65.9 %.
* **Cause qualitative** : rapport `EFR-510088` (filets cisaillés, traces d'intrusion → enlèvement / braconnage).
* **Contrôle environnemental** : `environmental_monthly` **innocente le climat** pour ce même site/mois (section 5.5).
* **Contrôle capteurs** : `monitoring_device_operations` isole ~4 % d'opérations en panne / maintenance lourde (section 4) — piste technique distincte de l'incident Kestrel (pas la cause de la chute).

### Fichiers encore à croiser / auditer
- `monitoring_sites.csv` : géographie KNT-KES (accès, vulnérabilité)
- `orchard_surveys.csv` : témoins périphérie (pistes de fuite éventuelles)

---

**Dossier écologie : enquête collaborative — braconnage KNT-KES établi (bat ↔ field reports), conforté par l'innocence climatique et le contrôle capteurs.**
