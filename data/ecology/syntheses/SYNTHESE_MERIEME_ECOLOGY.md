# Contribution Merieme — anomalies des sites et des vergers

État vérifié le 25 octobre 2026. Complément autonome à la synthèse collective, qui reste inchangée.

## Organisation et reproduction

- `../monitoring_sites_anomalies.ipynb` : 64 sites, 4 figures enregistrées.
- `../orchard_surveys_anomalies.ipynb` : 1 248 relevés, 6 figures enregistrées.
- Sources : `../cleaned/monitoring_sites_cleaned.csv` et `../cleaned/orchard_surveys_cleaned.csv`.
- Les notebooks de nettoyage existants sont conservés ; les nouvelles analyses ne réécrivent aucun CSV brut ou nettoyé.
- Lancer les cellules dans l'ordre depuis la racine du dépôt ou le dossier du notebook. Dépendances : pandas, numpy, scipy, matplotlib, scikit-learn, IPython/Jupyter.
- Exports : `<dataset>_anomalies.csv` et `<dataset>_seuils.csv`, dans ce dossier `syntheses/`. Une ligne d'anomalie correspond à une **méthode × variable × observation** ; plusieurs lignes peuvent concerner le même site ou relevé.

## Méthodes observées dans le groupe et choix retenus

Le code de `bat_surveys.ipynb` et `environmental_monthly.ipynb` utilise IQR, Z-score, anomalies contextuelles, Isolation Forest et LOF ; `health/patterns_anomalies.ipynb` utilise aussi le Z-score modifié et DBSCAN. Les notebooks logistiques `inventory_month_end` et `purchase_order_lines` utilisent IQR/Z puis IF/LOF. Les analyses du groupe montrent aussi K-Means, CAH, PCA, corrélations et Apriori.

Les synthèses sont utiles pour comprendre les questions du groupe, mais ne remplacent pas le code effectivement présent : par exemple, la synthèse ecology décrit davantage d'étapes pour `monitoring_device_operations` que son notebook actuel. Aucun de leurs verdicts causaux n'est utilisé comme étiquette dans ces analyses.

Les consignes du notebook de base demandent des titres et une explication du raisonnement, des résultats et des conclusions. Les nouveaux notebooks reprennent ce format.

| Analyse | Choix et justification |
|---|---|
| Sites | Distances Haversine en km ; IQR/Z/Z modifié par région ; LOF régional, k=5 et seuil 1,5, sensibilité k=3 et 8. Pas de quota de sites « anormaux ». |
| Vergers | Rendement mensuel en kg/ha ; référence saisonnière par site et mois calendaire, en excluant la ligne testée ; IQR/Z/Z modifié sur résidus et différences mensuelles de résidus ; IF/LOF sur six résidus standardisés par site. |
| Seuils | IQR 1,5 ; Z 3 ; Z modifié 3,5 ; groupes de moins de 8 valeurs ou dispersion nulle non évaluables. Pas de détection forcée lorsque MAD/IQR=0. |
| IF/LOF vergers | IF 200 arbres, seed 42, contamination 2 % ; LOF 20 voisins, contamination 2 %. Sensibilité : contamination 1/2/5 % et voisins 10/20/35. |

DBSCAN global isolerait surtout les régions pour les sites ; il n'est pas ajouté automatiquement. Clustering et Apriori ne sont pas nécessaires pour répondre aux questions ciblées de cette contribution.

## 1. Monitoring sites

### Audit et description

64 sites et aucune suppression. Une altitude manque (`KNT-CYP`) ; les 64 dates de fin sont absentes, compatibles avec des programmes actifs. Les catégories sont décrites par région, habitat et type de suivi.

- Aucune coordonnée hors des limites physiques ou manquante.
- Aucun couple latitude/longitude exactement dupliqué.
- Aucun site plus proche du centre d'une autre région que du sien : simple contrôle interne, pas validation de frontières administratives.
- Les cartes restent schématiques : les lieux sont fictifs, aucun géocodage externe n'est utilisé.

### Signaux

7 sites sont signalés par au moins une méthode statistique régionale ; LOF k=5 en signale 5. Leur union est de **8 sites**, pour 20 détections méthode/variable.

| Exemple | Mesure | Lecture |
|---|---|---|
| `AVL-ARU` | Voisin régional le plus proche à 24,77 km ; Z ≈ 3,28 | Isolement relatif, pas coordonnée impossible. |
| `KNT-YRX` | Voisin à 30,41 km ; LOF ≈ 3,73 | Site périphérique/isolé à examiner. |
| `KNT-CHP` | Altitude 418,8 m ; borne IQR haute 417,2 m | Dépassement très faible (1,6 m), faible argument pour une erreur. |
| `TMB-UVF` | Voisin à 23,13 km ; LOF ≈ 2,82 | Isolement géographique relatif. |

LOF k=5 signale `KNT-YRX`, `KNT-CHP`, `KNT-TEX`, `TMB-CKU`, `TMB-UVF`. Certains scores sont proches de 1,5 : consulter la sensibilité au voisinage avant d'interpréter ces cas.

**Conclusion :** des sites localement inhabituels existent, mais aucune erreur physique de coordonnées n'est démontrée. Vérifier l'intention d'échantillonnage, le relief et les sites isolés avant toute correction.

## 2. Orchard surveys

### Audit et contexte

16 sites × 78 mois, janvier 2019–juin 2025, clés site/mois uniques et séries mensuelles continues. Superficies positives, taux dans [0,100], pas de mesure numérique manquante. Les 1 225 notes vides restent sans interprétation automatique.

Le rendement est en **kg/ha par relevé mensuel**. Les comparaisons entre années utilisent également janvier–juin pour chaque année, afin de ne pas comparer 2025 à une année complète. Les huit taux convertis depuis des proportions au nettoyage restent une hypothèse de format héritée à valider si une source plus précise existe.

### Signaux

- 35 relevés signalés par les tests contextuels IQR/Z/Z modifié.
- 25 par IF, 25 par LOF, dont **12 communs** ; ce nombre dépend du quota de 2 %.
- Union de tous les indicateurs : **64 relevés**, pour 106 détections méthode/variable. Ce n'est pas une estimation du nombre de « vraies erreurs ».
- Plus forte baisse annuelle relative : **`ORC-0000812`, AVL-HUA, août 2021 : −54,55 %** du rendement par hectare par rapport à août 2020.
- `ORC-0000449`, AVL-GRB, novembre 2023 : fructification 28,7 %, résidu local avec Z ≈ −3,27.
- `ORC-0000532`, AVL-ECJ, avril 2024 : rendement ≈ 1 553,55 kg/ha, résidu local avec Z ≈ +3,14.

**Conclusion :** les relevés signalés sont des baisses, hausses ou combinaisons inhabituelles dans leur contexte local. Les données seules ne permettent pas d'attribuer ces variations à un prélèvement de chauves-souris, aux pesticides ou à une autre cause. Les notes et séries complètes doivent être relues.

## Limites et vérifications manuelles

1. Les références saisonnières reposent sur 6–7 années et utilisent rétrospectivement d'autres années : ce n'est pas une prévision prospective.
2. Les seuils univariés sont exploratoires ; les observations mensuelles et les méthodes sont dépendantes. Une accumulation de drapeaux n'est pas une accumulation de preuves indépendantes.
3. L'intersection IF/LOF est un accord, pas une confirmation. Les identifiants, noms et dates n'entrent pas comme valeurs numériques dans les modèles.
4. Les petits groupes régionaux et la sensibilité de LOF nécessitent une lecture des cartes, pas une correction automatique.
5. Aucune ligne signalée n'est supprimée et aucune cause de l'énigme n'est déclarée établie.

## Vérification technique et transparence

Les deux notebooks ont été exécutés intégralement avec des noyaux Jupyter neufs, depuis leur dossier, avec tableaux et images intégrés. Versions de vérification : Python 3.9, pandas 2.3.3, numpy 2.0.2, scipy 1.13.1, scikit-learn 1.6.1, matplotlib 3.9.4. Les paramètres et versions sont également affichés à l'exécution.



## Complément du 10 novembre 2026 — vérifications inter-sources

**Couverture du référentiel.** Les 16 sites disposant de relevés de vergers sont tous présents parmi les 64 sites de `monitoring_sites` et correspondent exactement aux 16 sites renseignés comme `commercial or research orchard` ; aucun identifiant de verger orphelin. Ce contrôle n'établit pas la justesse des coordonnées ni l'exhaustivité du réseau.

**Rapprochement exploratoire vergers / chauves-souris.** Jointure sur la clé composée `(site_code, mois)` après agrégation des visites de chauves-souris : **1 248 / 1 248 relevés** appariés, sur **16 sites**. Pour chacun des deux fichiers, les variations sont exprimées relativement à la médiane *du même site et du même mois calendaire dans les autres années* (leave-one-year-out). L'indicateur des chauves-souris est `observed_bat_count / observation_hours`, et non le seul nombre observé.

| Mesures résiduelles comparées | Spearman ρ | Intervalle bootstrap exploratoire 95 % (rééchantillonnage des 16 sites, 400 répétitions, graine 42) |
|---|---:|---:|
| Rendement (kg/ha) ↔ chauves-souris / heure | 0,146 | [0,093 ; 0,199] |
| Fructification (%) ↔ chauves-souris / heure | 0,195 | [0,130 ; 0,254] |

**Interprétation.** Il existe une association positive **modeste** dans ces données synthétiques, sans identification d'une cause : saisonnalité imparfaitement contrôlée, autocorrélation des séries, facteurs agricoles et environnementaux non inclus, variation de la détectabilité des chauves-souris. Les intervalles bootstrap décrivent la variabilité par site sous leurs hypothèses, **pas un intervalle causal** ; l'échantillon ne comporte que 16 sites. Le tableau peut soutenir une piste de discussion avec le groupe, pas une démonstration sur la pollinisation. Aucune ligne ni mesure n'a été modifiée pour obtenir ce résultat.
