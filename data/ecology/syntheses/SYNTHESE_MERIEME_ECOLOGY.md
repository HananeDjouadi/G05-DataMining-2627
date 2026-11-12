# SYNTHÈSE ÉCOLOGIE — SITES DE SUIVI ET VERGERS

**Master 2 Data Science — Groupe 05 | Année 2026–2027**
**Fichiers étudiés :** `monitoring_sites` et `orchard_surveys`

---

## SYNTHÈSE EXÉCUTIVE

| Aspect | Résultat |
| :--- | :--- |
| **Données étudiées** | 64 sites de suivi et 1 248 relevés mensuels de vergers (16 sites) |
| **Bloc 2 — Clustering** | K-Means, CAH (Ward), dendrogrammes et visualisations ACP ; t-SNE en comparaison |
| **Exploration des vergers** | Pearson/Spearman, nuages de points et décomposition saisonnière STL |
| **Bloc 3 — Anomalies** | IQR, Z-score, Z-score modifié, Isolation Forest, LOF, DBSCAN et règles Apriori |
| **Monitoring sites** | 8 sites distincts signalés par les méthodes statistiques et LOF |
| **Orchard surveys** | 64 relevés distincts signalés par les méthodes statistiques, Isolation Forest ou LOF |
| **Croisement qualitatif** | 3 relevés atypiques comportent également une note de perturbation agricole |
| **Conclusion principale** | Des sites et des relevés méritent une vérification, mais les analyses ne prouvent pas à elles seules une erreur ou sa cause |

---

## Sommaire

1. [Monitoring sites — Localisation des sites](#1-monitoring-sites--localisation-des-sites)
2. [Orchard surveys — Relevés des vergers](#2-orchard-surveys--relevés-des-vergers)
3. [Connexion entre les deux fichiers](#3-connexion-entre-les-deux-fichiers)
4. [Bilan transversal](#4-bilan-transversal)

---

## 1. MONITORING SITES — LOCALISATION DES SITES

### 1.1 Prétraitement et exploration

Le fichier contient **64 sites**, répartis entre trois régions fictives : **22** dans la région de Kintara, **22** en Avelora et **20** à Tembara.

- **Doublons :** aucun doublon exact ni doublon de `site_code`.
- **Coordonnées :** aucune latitude ou longitude manquante, hors limites ou strictement dupliquée.
- **Valeurs manquantes :** une altitude absente (`KNT-CYP`) et 64 dates de fin de programme non renseignées.
- **Distances :** calcul de la distance au centre régional et au voisin régional le plus proche avec la formule de **Haversine**.

**Choix :** les distances sont analysées **par région**. Comparer directement des sites de régions éloignées créerait des anomalies artificielles.

### 1.2 Bloc 2 — Clustering et réduction de dimension

**Méthodes utilisées :** standardisation, encodage des catégories, K-Means, CAH Ward, dendrogramme, ACP et t-SNE exploratoire.

| Méthode | Résultat | Interprétation |
| :--- | :--- | :--- |
| **K-Means** | **5 groupes** retenus parmi les valeurs de `k` testées | Profils de sites présentant des caractéristiques similaires |
| **CAH (Ward)** | Dendrogramme et partition comparée à K-Means | Permet d'observer les regroupements hiérarchiques |
| **Accord K-Means / CAH** | **ARI = 0,668** | Accord partiel entre les deux partitions |
| **ACP / t-SNE** | Projections en deux dimensions | Illustrent les profils, sans prouver une anomalie |

Les variables de regroupement associent les distances régionales, l'altitude et des catégories de sites. Le regroupement reste **descriptif** : deux sites semblables n'ont pas nécessairement le même rôle écologique.

### 1.3 Bloc 3 — Détection d'anomalies

**Détection univariée :** IQR, Z-score et Z-score modifié, appliqués par région aux distances et à l'altitude. **Détection locale :** LOF sur les coordonnées avec distance de Haversine et sensibilité à différents nombres de voisins.

| Résultat | Nombre |
| :--- | ---: |
| Sites signalés par au moins un test univarié | **7** |
| Sites signalés par LOF (`k = 5`) | **5** |
| **Union des sites signalés** | **8** |
| Détections totales (méthode × variable × site) | **20** |

Les méthodes supplémentaires **Isolation Forest** et **DBSCAN** servent à comparer les alertes spatiales. Elles ne sont pas incluses dans le chiffre de 8 sites issu du premier bilan.

**Exemples à examiner :**

| Site | Observation | Lecture |
| :--- | :--- | :--- |
| `KNT-YRX` | Voisin régional à environ **30,41 km**, LOF ≈ **3,73** | Site relativement isolé |
| `KNT-CHP` | Altitude **418,8 m**, légèrement au-dessus de la borne IQR ≈ **417,2 m** | Dépassement faible ; relief à vérifier |
| `AVL-ARU` | Voisin régional à environ **24,77 km**, Z-score ≈ **3,28** | Distance inhabituelle dans sa région |
| `TMB-UVF` | Voisin régional à environ **23,13 km**, LOF ≈ **2,82** | Faible densité locale |

**Règles d'association (Apriori) :** les catégories de sites, les habitats, les accès et le contexte des vergers sont étudiés ensemble. **49 ensembles fréquents** et **93 règles** respectent les seuils choisis. Plusieurs associations traduisent simplement la façon dont les catégories ont été définies ; elles ne constituent pas une découverte causale.

### 1.4 Graphe de proximité et contrôles croisés

Un graphe relie deux sites d'une même région lorsqu'ils sont suffisamment proches.

- **Seuil de 25 km :** Kintara possède **3 composantes**, dont deux sites isolés (`KNT-CHP` et `KNT-YRX`).
- **Seuil de 35 km :** les sites deviennent connectés à l'intérieur de chacune des trois régions.
- **Seuil de 15 km :** les réseaux sont davantage fragmentés.

**Conclusion monitoring sites :** certaines localisations sont atypiques au regard de leur région, mais **aucune erreur de coordonnées n'est démontrée**. Un site isolé peut résulter d'un choix d'échantillonnage.

---

## 2. ORCHARD SURVEYS — RELEVÉS DES VERGERS

### 2.1 Prétraitement et exploration

Le fichier comprend **1 248 relevés**, soit **16 vergers suivis pendant 78 mois**, de janvier 2019 à juin 2025.

- **Clés :** un relevé unique par site et par mois ; aucune série mensuelle interrompue.
- **Variables étudiées :** rendement ramené à l'hectare, fructification, activité des autres pollinisateurs, fertilisation et irrigation.
- **Valeurs manquantes :** pas de mesures numériques manquantes dans les variables analysées ; **1 225 notes agricoles non renseignées**.
- **Saisonnalité :** comparaison de chaque relevé à l'historique du **même site et du même mois**, en excluant l'observation examinée.

**Choix :** il ne serait pas pertinent de comparer directement un rendement de janvier avec celui d'un mois de récolte, ni le début de 2025 avec une année complète.

**Corrélations et graphiques (EDA) :** les matrices de Pearson et Spearman et les nuages de points comparent rendement, fructification, densité florale, activité des autres pollinisateurs, fertilisation et irrigation. Le rendement est fortement lié à la fructification (**Pearson 0,915 ; Spearman 0,908**), plus modérément à l'activité des autres pollinisateurs (**0,427 ; 0,401**). Ce sont des associations globales : les différences entre sites et saisons peuvent intervenir, sans qu'une cause soit établie.

**STL (analyse temporelle) :** une décomposition sur les **78 mois du verger `AVL-ECJ`**, choisi car son rendement moyen est proche de la médiane des 16 vergers, sépare tendance, saisonnalité annuelle et résidus. Le résidu absolu le plus élevé est observé en **avril 2024 (≈ +508,56 kg/ha)**. Ce résultat sert de visualisation complémentaire : un résidu élevé ne prouve pas une erreur ni une cause écologique.

### 2.2 Bloc 2 — Clustering et réduction de dimension

**Méthodes utilisées :** standardisation des profils de vergers, ACP, K-Means, CAH Ward et dendrogramme. Une projection t-SNE et DBSCAN sont ajoutés à titre exploratoire.

| Méthode | Résultat | Interprétation |
| :--- | :--- | :--- |
| **ACP** | Deux premières composantes : **60,6 % + 18,9 %** de variance | Environ **79,5 %** de l'information statistique représentée sur deux axes |
| **K-Means** | **2 groupes de 8 vergers** | Choix évitant les groupes d'un seul site |
| **CAH (Ward)** | Dendrogramme des **16 vergers** | Autre façon de comparer les profils |
| **Accord K-Means / CAH** | **ARI = 0,749** | Regroupements assez proches, sans être identiques |

Les profils sont construits à partir de six caractéristiques : niveaux moyens et variabilité du rendement et de la fructification, activité des autres pollinisateurs et utilisation des fertilisants.

**Lecture :** les groupes décrivent des comportements moyens différents. Ils n'expliquent pas, à eux seuls, les changements soudains observés certains mois.

### 2.3 Bloc 3 — Détection d'anomalies

**Détection univariée :** IQR, Z-score et Z-score modifié sur les écarts aux références saisonnières. **Détection multivariée :** Isolation Forest et LOF sur les résidus standardisés par site.

| Méthode | Relevés signalés |
| :--- | ---: |
| Tests statistiques contextuels | **35** |
| Isolation Forest (contamination = 2 %) | **25** |
| LOF (contamination = 2 %) | **25** |
| Accord Isolation Forest ∩ LOF | **12** |
| **Union de toutes les méthodes** | **64 relevés distincts** |

Le tableau détaillé des résultats comprend **106 détections** : un même relevé peut être signalé par plusieurs variables ou méthodes.

**Exemples à examiner :**

| Relevé | Signal | Lecture |
| :--- | :--- | :--- |
| `ORC-0000812` — `AVL-HUA` | Baisse annuelle du rendement de **54,55 %** (août 2021) | Forte variation par rapport à août 2020 |
| `ORC-0000449` — `AVL-GRB` | Fructification **28,7 %**, Z local ≈ **−3,27** | Relevé faible par rapport au contexte du site |
| `ORC-0000532` — `AVL-ECJ` | Rendement ≈ **1 553,55 kg/ha**, Z local ≈ **+3,14** | Relevé élevé par rapport au contexte du site |

### 2.4 Patterns fréquents et croisement qualitatif

**Apriori :** paniers construits à partir des saisons et de catégories dérivées des rendements, de la fructification, des autres pollinisateurs et de la présence d'une note agricole. Le calcul produit **97 ensembles fréquents** et **40 règles** aux seuils retenus.

Les règles les plus visibles associent fréquemment **rendement faible** et **fructification faible**. Ce rapprochement est descriptif et attendu : il ne démontre pas l'existence d'une cause extérieure.

**Croisement avec les notes de perturbation :**

| Situation | Avec note | Sans note |
| :--- | ---: | ---: |
| Relevés signalés par au moins une méthode | **3** | **61** |
| Relevés non signalés | **20** | **1 164** |

Les **3 relevés atypiques disposant d'une note agricole** concernent le site `AVL-FNQ` :

- `ORC-0000945` : application élevée de pesticides, septembre 2019.
- `ORC-0000947` : pertes de fleurs liées à une tempête, novembre 2019.
- `ORC-0000965` : application élevée de pesticides, mai 2021.

**Conclusion orchard surveys :** ces notes constituent des pistes de vérification, **pas une preuve que les événements notés ont causé les anomalies**.

---

## 3. CONNEXION ENTRE LES DEUX FICHIERS

### 3.1 Couverture des sites

Les **16 sites** présents dans `orchard_surveys` sont tous référencés dans `monitoring_sites`. Ils correspondent aux **16 sites décrits dans un contexte de verger** : aucun identifiant n'est orphelin.

### 3.2 Complément avec `bat_surveys`

Les relevés de vergers ont été rapprochés des observations de chauves-souris par **site et mois** : **1 248 relevés appariés**, sur **16 sites**. Les comparaisons portent sur les écarts aux références saisonnières, et les observations de chauves-souris sont ramenées à une durée d'observation.

| Indicateurs comparés | Corrélation de Spearman (ρ) | Intervalle bootstrap exploratoire 95 % |
| :--- | ---: | :--- |
| Rendement ↔ chauves-souris par heure | **0,146** | [0,093 ; 0,199] |
| Fructification ↔ chauves-souris par heure | **0,195** | [0,130 ; 0,254] |

**Lecture :** les associations sont **positives mais modestes**. Elles ne prouvent pas que l'activité des chauves-souris provoque les variations de rendement. La saisonnalité, les conditions agricoles et les autres facteurs environnementaux peuvent intervenir.

---

## 4. BILAN TRANSVERSAL

| Question | Réponse apportée par les analyses |
| :--- | :--- |
| Les coordonnées des sites sont-elles incohérentes ? | **Non, aucune incohérence physique démontrée.** Quelques sites sont relativement isolés. |
| Les vergers présentent-ils des valeurs atypiques ? | **Oui, 64 relevés distincts** sont signalés par au moins une des méthodes principales. |
| Les méthodes de clustering sont-elles concordantes ? | Accord K-Means / CAH **partiel à assez marqué** selon le fichier (ARI **0,668** et **0,749**). |
| Les notes agricoles expliquent-elles les anomalies ? | **Non démontré.** Trois relevés atypiques comportent une note, à examiner. |
| Les observations de chauves-souris permettent-elles une conclusion causale ? | **Non.** Les corrélations observées sont modestes et descriptives. |

### Limites et suite de l'enquête

- Les régions et sites sont **fictifs** ; les cartes ne valident pas des localisations réelles.
- Les seuils des méthodes d'anomalies et les paramètres de clustering influencent les résultats.
- Les détections d'un même relevé ne sont **pas des preuves indépendantes**.
- Les séries mensuelles et les 16 sites de vergers ne permettent pas, à eux seuls, de déterminer une cause.
- **Aucune ligne signalée n'est supprimée.** Les anomalies sont proposées à la discussion et à la vérification avec les autres fichiers Ecology.

**Conclusion générale :** le référentiel des sites est cohérent mais comporte des localisations peu représentées. Les relevés des vergers montrent des variations saisonnières et quelques observations particulièrement atypiques. Les croisements avec les notes agricoles et `bat_surveys` orientent l'enquête, sans suffire à confirmer une explication causale.

---
