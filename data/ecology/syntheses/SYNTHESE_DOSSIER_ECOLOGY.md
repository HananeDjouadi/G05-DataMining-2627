# 🌿 Dossier Thématique : Écologie & Surveillance Faunique (`data/ecology/`)
**Projet de Data Mining — Master 2 Data Science & Intelligence Artificielle (Groupe 05)**  
*Auteurs : Équipe G05 | Année Universitaire 2026-2027*

---

## 📌 Sommaire du Dossier
1. [Vue d'ensemble du domaine écologique & Problématique](#1-vue-densemble-du-domaine-écologique)
2. [Fiche 1 : Analyse quantitative des chauves-souris (`bat_surveys.ipynb`)](#2-fiche-1--analyse-quantitative-des-chauves-souris-bat_surveysipynb)
   - [2.1 Prétraitement et Nettoyage des données](#21-prétraitement-et-nettoyage-des-données)
   - [2.2 Bloc 1 : EDA et Statistiques Descriptives](#22-bloc-1--eda-et-statistiques-descriptives)
   - [2.3 Bloc 2 : Clustering et Réduction de Dimensionnalité](#23-bloc-2--clustering-et-réduction-de-dimensionnalité)
   - [2.4 Bloc 3 : Détection d'Anomalies, Patterns & Enquête](#24-bloc-3--détection-danomalies-patterns--enquête)
   - [2.5 Pièces à conviction & Bilan Médico-Légal](#25-pièces-à-conviction--bilan-médico-légal)
3. [Fiche 2 : Analyse textuelle des rapports de terrain (`environmental_field_reports.ipynb`)](#3-fiche-2--analyse-textuelle-des-rapports-de-terrain-environmental_field_reportsipynb) *(En cours)*
4. [Fiche 3 : Analyse des opérations de capteurs (`monitoring_device_operations.ipynb`)](#4-fiche-3--analyse-des-opérations-de-capteurs-monitoring_device_operationsipynb) *(À venir)*
5. [Fiche 4 : Contrôle environnemental mensuel (`environmental_monthly.ipynb`)](#5-fiche-4--contrôle-environnemental-mensuel-environmental_monthlyipynb)
    - [5.1 Prétraitement et Nettoyage des données](#51-prétraitement-et-nettoyage-des-données)
    - [5.2 Bloc 1 : EDA et Statistiques Descriptives](#52-bloc-1--eda-et-statistiques-descriptives)
    - [5.3 Bloc 2 : Clustering et Réduction de Dimensionnalité](#53-bloc-2--clustering-et-réduction-de-dimensionnalité)
    - [5.4 Bloc 3 : Détection d'Anomalies, Patterns & Enquête](#54-bloc-3--détection-danomalies-patterns--enquête)
    - [5.5 Contrôle croisé KNT-KES & Bilan](#55-contrôle-croisé-knt-kes--bilan)
6. [Synthèse Transversale & Résolution de l'Enquête du Dossier](#6-synthèse-transversale--résolution-de-lenquête-du-dossier)

---

## 1. Vue d'ensemble du domaine écologique

Le dossier `data/ecology/` regroupe l'ensemble des données de surveillance environnementale menées sur des réserves naturelles et grottes karstiques sensibles abritant des espèces vulnérables de chauves-souris (*Eonycteris spelaea*, *Rhinolophus ferrumequinum*).

Ce système de surveillance combine :
* **Des relevés écologiques quantitatifs** (`bat_surveys.csv`) : suivis des colonies, comptages visuels, capteurs acoustiques ultrasons.
* **Des rapports qualitatifs de gardes forestiers** (`environmental_field_reports.csv`) : observations textuelles de terrain et signalements d'incidents.
* **Des logs opérationnels d'équipements** (`monitoring_device_operations.csv`) : statut fonctionnel et anomalies sur les stations de surveillance.
* **Des historiques climatiques mensuels** (`environmental_monthly.csv`) : précipitations, humidité et températures.

---

## 2. Fiche 1 : Analyse quantitative des chauves-souris (`bat_surveys.ipynb`)

### 2.1 Prétraitement et Nettoyage des données

* **Volumétrie initiale** : 4 994 relevés d'observation répartis de janvier 2019 à mai 2025.
* **Justifications de nos choix méthodologiques** :
  1. **Doublons stricts** : 2 lignes strictement identiques identifiées et supprimées $\implies$ taille finale de **4 992 relevés**.
  2. **Gestion des valeurs manquantes** : Des valeurs nulles étaient présentes sur l'équipe observatrice (`observer_team`). Au lieu de supprimer ces lignes (ce qui aurait brisé la continuité des séries temporelles), nous avons **imputé par le mode historique propre à chaque site**, garantissant le respect de l'affectation géographique des équipes.
  3. **Normalisation des formats** : Standardisation des dates (`survey_start`), suppression des espaces parasites (`" count"`) et conversion des séparateurs décimaux en virgules vers des points flottants.

---

### 2.2 Bloc 1 : EDA et Statistiques Descriptives

* **Distributions biologiques** :
  * Les variables quantitatives (`colony_estimate`, `observed_bat_count`, `acoustic_detection_count`) présentent une **forte asymétrie à droite (log-normale/exponentielle)**. Cette structure est naturelle en écologie : la majorité des colonies sont modestes, tandis que quelques grottes majeures abritent de très fortes densités.
* **Corrélations clés** :
  * Corrélation de Pearson très élevée ($r > 0.85$) entre les détections acoustiques ultrasons et les comptages visuels, validant la cohérence croisée des dispositifs de capture.
  * Corrélation négative marquée avec l'interférence météo (`weather_interference`) : les épisodes pluvieux et venteux contraignent les chiroptères à demeurer au gîte.
* **Saisonnalité** :
  * Maximum d'activité biologique enregistré en **été** (juin-août, période de mise bas et de lactation).
  * Minimum biologique en **hiver** (décembre-février, période de léthargie/hibernation).

---

### 2.3 Bloc 2 : Clustering et Réduction de Dimensionnalité

* **Standardisation (`StandardScaler`)** :  
  Indispensable car les échelles variaient entre des estimations de colonies en milliers et des indices de qualité ordonnés de 1 à 3.
* **Choix du nombre de clusters ($k=3$)** :
  * Méthode du Coude (*Elbow*) : Cassure nette de l'inertie intra-cluster à $k=3$.
  * Score de Silhouette : Pic de séparation à **$0.1952$**, confirmant une partition compacte.
  * Dendrogramme hiérarchique (CAH, méthode de Ward) : La coupe à distance 15 confirme rigoureusement 3 branches mères stables.
* **Typologie des 3 profils identifiés** :
  * 🟢 **Cluster 0 (939 relevés, ~19%) — Mégas-colonies estivales** : Activité acoustique et visuelle maximale, conditions idéales en période de reproduction.
  * 🔵 **Cluster 1 (1 843 relevés, ~37%) — Activité perturbée / Transition** : Activité modérée associée à des conditions météorologiques sévères.
  * 🔴 **Cluster 2 (2 210 relevés, ~44%) — Basse activité standard** : Relevés d'intersaison et d'hiver avec faible émergence.
* **Validation spatiale (PCA 2D)** :  
  La projection sur les 2 premières composantes principales capture **67.2% de la variance totale** et affiche une séparation nette sans chevauchement pathologique.

---

### 2.4 Bloc 3 : Détection d'Anomalies, Patterns & Enquête

#### A. Détection Univariée (IQR & Z-Score)
* L'IQR global isole les records hauts mondiaux (`colony_estimate` > 37.30, 155 points).
* **Limite méthodologique démontrée** : La borne basse de l'IQR étant négative ($-7.50$), les méthodes univariées globales sont **totalement aveugles aux chutes anormales de population** survenant sur des sites historiquement denses.

#### B. Détection Multivariée (Isolation Forest & LOF)
* Modèles paramétrés avec une contamination de 2% (100 anomalies isolées).
* Les modèles isolent les points géométriquement extrêmes (ex: méga-site `TMB-BRC` avec 93.7 individus).
* **Constat clé** : Un modèle multivarié non supervisé global est aveugle au contexte temporel et local. D'où la nécessité de passer au Z-score contextuel conditionnel.

![Projection PCA 2D et Détection Multivariée d'Anomalies](images/pca_anomalies.png)
*Figure 1 : Projection PCA à 2D des relevés. En rouge : anomalies globales détectées par Isolation Forest. Étoile jaune au centre : relevé BAT-0000064, indétectable par un modèle global car entouré de relevés d'hiver ordinaires.*

#### C. Extraction de Règles d'Association (`mlxtend` / Apriori)
Application de l'algorithme Apriori sur les données discrétisées (selon les Slides 4 à 11 du Cours 3) :
* **Règle n°1** :  
  $$\text{SI } \{\text{Météo Sévère, Qualité 2}\} \implies \text{ALORS } \{\text{Cluster 1}\} \quad (\mathbf{Support = 17.9\%}, \mathbf{Confiance = 89.3\%}, \mathbf{Lift = 2.42})$$
* **Interprétation** : Le mauvais temps contraint quasi-systématiquement la mission à basculer dans le profil d'activité perturbée, avec un ratio de présence 2.42 fois supérieur au hasard.

---

### 2.5 Pièces à conviction & Bilan Médico-Légal

![Série Temporelle KNT-KES et Effondrement de la Colonie](images/knt_time_series_crash.png)
*Figure 2 : Évolution chronologique de la taille de la colonie à KNT-KES (2019-2025). Le point rouge met en évidence la chute vertigineuse de -65.9% le 5 avril 2024 (BAT-0000064).*

| Indicateur | Valeur Normale Historique | Relevé Suspect `BAT-0000064` | Écart / Signature |
| :--- | :--- | :--- | :--- |
| **Site géographique** | Grotte `KNT-KES` (Kestrel) | Grotte `KNT-KES` | Épicentre du sinistre |
| **Date du relevé** | Avril (mois 4) | **05 Avril 2024 (18h00)** | Horodatage de l'incident |
| **Taille de la colonie** | 27.9 individus (Avril 2023) | **9.5 individus (Avril 2024)** | **Effondrement brutal de -65.9%** |
| **Interférence météo** | Variable | **29.9 (Normale)** | Écarte toute cause climatique |
| **Z-Score Contextuel** | $\approx 0$ | **$Z < -3$ (Anomalie locale)** | Déviation extrême par rapport au site |

> **Conclusion Forensic :**  
> L'effondrement quantitatif mesuré sur le relevé `BAT-0000064` est la signature directe du raid clandestin documenté dans le rapport de terrain **`EFR-510088`** (filets anti-intrusion cisaillés, vol massif de chauves-souris vivantes de l'espèce protégée *Eonycteris spelaea* pour alimenter des filières illicites d'expérimentation biologique).

---

## 3. Fiche 2 : Analyse textuelle des rapports de terrain (`environmental_field_reports.ipynb`)
*(En cours de traitement)*

---

## 4. Fiche 3 : Analyse des opérations de capteurs (`monitoring_device_operations.ipynb`)
*(À venir)*

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

## 6. Synthèse Transversale & Résolution de l'Enquête du Dossier
*(En consolidation — éléments déjà acquis :)*

* **Épicentre confirmé** : `BAT-0000064` @ `KNT-KES` (05/04/2024) — chute de colonie −65.9 %.
* **Cause qualitative** : rapport `EFR-510088` (filets cisaillés, traces d'intrusion → enlèvement / braconnage).
* **Contrôle environnemental** : `environmental_monthly` **innocente le climat** pour ce même site/mois (section 5.5).
* *Reste à intégrer :* `environmental_field_reports` (fiche 2), `monitoring_device_operations` (fiche 3), et croisement final des 4 sources.
