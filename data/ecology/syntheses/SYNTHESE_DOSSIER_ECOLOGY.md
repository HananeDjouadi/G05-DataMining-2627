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
5. [Fiche 4 : Contrôle environnemental mensuel (`environmental_monthly.ipynb`)](#5-fiche-4--contrôle-environnemental-mensuel-environmental_monthlyipynb) *(À venir)*
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
*(À venir)*

---

## 6. Synthèse Transversale & Résolution de l'Enquête du Dossier
*(Sera consolidée à l'issue du traitement des 4 fichiers du dossier)*
