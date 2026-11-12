# Synthèse — Dossier Health, partie clinique

*Groupe G05 · Tables : `clinical_encounters`, `clinical_notes`, `clinical_observation`, `clinical_procedures`, `patient_administration`*

*Notebooks (dans l'ordre) : `cleaning.ipynb` → `eda.ipynb` (bloc 1) → `clustering.ipynb` (bloc 2) → `patterns_anomalies.ipynb` (bloc 3) → `graphs_process.ipynb` (bloc 4) → `spatio_temporal.ipynb` (bloc 5) → `text_mining.ipynb` (bloc 6) → `narrative.ipynb` (bloc 7).*

---

## 1. Nettoyage

- **Doublons exacts** : 6 supprimés (encounters 2, observation 2, procedures 1, patient_administration 1).
- **Dates** : 11 colonnes converties en `datetime` (formats mixtes). Aucune incohérence : pas de fin avant le début, aucune observation hors de sa consultation.
- **Types** : `numeric_value` (observation) contenait ~340 valeurs textuelles (virgule décimale, unité collée, ex. « 82.2 index »), extraites puis converties en `float`.
- **Catégories** : minuscules et espaces nettoyés (ex. `encounter_type` 15 → 10 modalités, `interpretation_code` 9 → 4). Identifiants, licences et codes de procédure laissés intacts.
- **Manquants** : colonne vide `superseded_document_id` supprimée ; absences codées explicitement (`no_device`, `no_material`, `unspecified`, `non_applicable`) ; dates manquantes laissées en `NaT`.
- **Aberrants** : aucune ligne supprimée, pour ne pas perdre d'indices. Sorties : `cleaned/*_cleaned.csv`.

## 2. Bloc 1 — Analyse exploratoire

**Méthode.** Typage de chaque variable (22 identifiants, 10 dates, 25 nominales dont 4 ordonnées, 2 textes libres, 1 numérique). Dates transformées en durées (consultation, délai de suivi, retard de procédure, délai note − consultation, âge). Résumés numériques (moyenne ± écart-type, médiane, quartiles), histogrammes et boxplots, test de normalité, CCDF log-log pour la loi de puissance. `numeric_value` décrite par type de mesure (27 types). Corrélations de Pearson et Spearman sur une table « une ligne par consultation ».

- **Lois** : durées et retards multimodaux ou dégénérés (pics à 180 / 240 min, retard de 10 min quasi constant) ; âge à l'inscription symétrique mais presque uniforme (15-80 ans) ; **pas de loi de puissance** (consultations par patient bornées, 3 à 11).
- **Corrélations** : apical / pouls périphérique (Spearman 0,67), âge biologique / inflammation (0,43), mobilité / vitesse de marche (0,71). Le nuage apical vs pouls montre une **sous-population décalée** : les consultations `discordant`. Corrélation ≠ causalité : apical et pouls ont une cause commune (le cœur).
- **Artefact** : 239 inscriptions sur 240 au **1er du mois**.

## 3. Bloc 2 — Clustering

**Méthode.** Dates, textes, identifiants et noms exclus : il ne reste que des variables catégorielles. Encodage one-hot, distance de Hamming (cours 2, variables nominales), classification hiérarchique (liaison `average`). K-Means (coude, silhouette) en comparaison. Visualisation par PCA / t-SNE.

| Table | k : tailles | Lecture | Silhouette |
|---|---|---|---|
| `clinical_encounters` (1 067) | 3 : 1 004 / 40 / 23 | Flux courant (suivi planifié 80 %) ; 40 consultations confidentielles (92 %), souvent clôturées ; 23 consultations de recherche (`research_procedure` 87 %, 100 % restreintes) | 0,16 |
| `clinical_notes` (30) | non clusterisée | 2 variables catégorielles, 30 lignes, tableau croisé sans structure ; l'information est dans le texte (bloc 3) | – |
| `clinical_observation` (17 734) | 4 : 11 250 / 5 188 / 1 017 / 279 | Mesures dans la norme ; observations qualitatives « recorded » ; fréquence apicale (1 017) ; mesures à revoir `review` (279) | 0,26 |
| `clinical_procedures` (310) | 3 : 260 / 30 / 20 | Procédures standard ; 30 `restricted_specialist` ; 20 `research_restricted` | 0,64 |
| `patient_administration` (240) | 4 : 193 / 38 / 7 / 2 | Dossiers actifs ; 38 `nonbinary` actifs ; 9 dossiers inactifs | 0,27 |

**À retenir**
- Seule `clinical_procedures` a une structure nette : 50 procédures isolées (code `MRP-42`), toutes restreintes, chacune avec un matériel distinct. K-Means sur les variables brutes (silhouette 0,63) isole exactement les mêmes 50.
- Ailleurs, pas de groupes naturels (silhouette < 0,3) : les clusters reflètent des combinaisons de modalités (type de mesure, sexe × statut), pas des profils cachés.

## 4. Bloc 3 — Patterns et anomalies

**Méthode.** Règles d'association (Apriori, `mlxtend`) sur les variables catégorielles ; anomalies univariées (IQR, z-score, z modifié) puis multivariées (Isolation Forest, LOF, DBSCAN) sur des variables numériques construites (durées, retards, délais, âges). Chaque anomalie est discutée : valeur rare mais légitime, ou artefact.

| Table | Patterns | Anomalies | Nature |
|---|---|---|---|
| `clinical_encounters` | 2 règles seulement, lift ≈ 1,2 : pas de structure | **50 consultations de durée fixe** (20 `research_procedure` à 180 min, 30 `specialist_procedure` à 240 min, toutes restreintes) ; 2 délais de suivi > 1 000 j | Signature de protocole, pas des erreurs |
| `clinical_notes` (30) | 1 règle fragile (`imaging_report` ⇒ `FAC-VLN-101`, lift 1,47) | **8 notes sur 30 datées avant leur consultation** (jusqu'à −1 249 j) ; CLN-510126 à +2 670 j ; longueurs homogènes | Incohérence de données |
| `clinical_observation` | `recorded` ⇒ `no_device` (lift 3,4) ; règles triviales sinon | **109 `discordant`** invisibles en univarié : écart apicale − pouls périphérique de 22,5 bpm contre 1,7. Isolation Forest en retrouve 17/109 (groupe trop dense : 10,7 %) | Anomalie contextuelle réelle |
| `clinical_procedures` | `research_restricted` (lift 15,5) et `restricted_specialist` (lift 10,3) liés à des issues propres à `MRP-42` | Retards constants de 35 et 40 min pour ces 50 procédures ; **2 retards extrêmes** : PRC-000054 (59 j) et PRC-000071 (177 j) | Signature de classe ; 2 vraies anomalies |
| `patient_administration` | Aucune règle (lift ≥ 1,2) | PAT-31099, PAT-31174, PAT-31183 isolés par les 3 méthodes : dossiers anciens (2014-2015), professions dirigeantes ou politiques | Rares mais plausibles |

**À retenir**
- Les anomalies univariées sont surtout des **constantes propres à un protocole** (durée, retard) : statistiquement aberrantes, mais pas des erreurs. IQR et MAD se dégradent quand la majorité des valeurs sont identiques (retards de 10 min) ; le z modifié avec repli sur l'écart moyen absolu isole alors les vrais extrêmes.
- La discordance apicale/périphérique est une anomalie **bivariée** : aucune valeur n'est extrême seule, et les méthodes non supervisées la manquent à 5 % de contamination.

## 5. Bloc 4 — Graphes et processus

**Méthode.** Graphe biparti patients – professionnels (G : 248 nœuds, 844 liens pondérés par le nombre d'interactions), projection professionnels (P, complète), sous-graphe des interactions **restreintes** (R : 116 nœuds, 165 liens). Degrés, CCDF, closeness et betweenness (distance = 1 / poids), plus courts chemins, Louvain, Girvan-Newman, Infomap (si `cdlib`), modularité. Process mining sur l'event log des consultations (cas = patient, activité = type) : DFG, mesure de dépendance du Heuristic Miner, variantes ; `pm4py` en option.

| Résultat | Valeur | Lecture |
|---|---|---|
| Loi de puissance | non (degrés bornés, structure en étoiles) | Patientèle ordinaire |
| Centralité dans G | degrés homogènes (96-118), mais `LIC-VLN-MED-1002` a le degré pondéré (229) et la betweenness (0,31) maximaux | Il voit certains patients beaucoup plus souvent |
| Centralité dans R | `LIC-VLN-MED-1002` : 39 voisins, degré pondéré 99 (suivant : 34), betweenness 0,50 | Hub des interactions restreintes |
| Liens les plus lourds de R | `LIC-VLN-MED-1002` ↔ les **8 patients `MRP-42 specialist`** (6 à 10 interactions chacun) | Circuit fermé |
| Louvain sur R | Q = 0,55 ; la communauté de 1002 contient les 8 patients `specialist` ; 14 des 20 patients « recherche » dispersés ailleurs | Deux organisations distinctes |
| Girvan-Newman sur R | Q = 0,40 ; non pondéré, il disperse les 8 patients | Le regroupement tient aux poids |
| Process mining | 212 patients « autres » : 120 variantes, dépendances ≈ 0 (spaghetti) ; 8 `specialist` : 3 variantes, cycle `executive_health_review` → `specialist_procedure` ; 20 « recherche » : 19 variantes, une seule `research_procedure` | Traitement récurrent vs administration unique |

## 6. Bloc 5 — Spatial et temporel

**Spatial.** Coordonnées seulement pour les 3 établissements (`public_records/licensed_facilities`) ; lieux indirects : `home_region` (5 régions fictives). Carte à taille variable, agrégation par site et par région, χ².
- Les 30 `MRP-42 specialist` ont lieu **uniquement** dans les deux cliniques Asterion (`FAC-VLN-101`, `FAC-AVL-201`), jamais au laboratoire `FAC-AVL-301`.
- Les 8 patients `specialist` viennent des **5 régions** : réseau transnational, pas recrutement local.
- Discordance indépendante de la région (χ² p = 0,36) et présente dans les 3 sites (0,5 à 0,9 % des observations).
- Moran, LISA, régionalisation, Voronoï : **non pertinents** (3 sites, 5 régions, coordonnées fictives).

**Temporel.** Séries mensuelles 2017-2025, ACF, décomposition classique et STL (`statsmodels`), anomalies par fenêtre glissante (z-score et z modifié sur 12 mois), intervalles entre événements pour les séries creuses.
- **Tendance** en cloche 2017 → 2021 → 2025 = entrée/sortie de la cohorte (inscriptions 2017-2021), **pas de saisonnalité** (ACF sans pic à 12 mois une fois la tendance retirée ; composante saisonnière < résidu).
- **Erreur évitée** : consultations et observations discordantes corrélées à 0,85 par année, uniquement à cause de la tendance commune (0,50 après différenciation ; le taux de discordance ne suit pas la cloche).
- **Anomalie temporelle = artefact** : 22 consultations le **2025-06-20**, presque toutes à 13 h 15, dans les 3 sites, avec des praticiens présents dans deux sites à la même heure ; ce sont les dernières visites de 22 patients et la date de suivi prévue à la visite précédente vaut exactement 2025-06-20 → dates de suivi **plafonnées** par le générateur.
- `MRP-42 specialist` : une administration tous les **~742 jours** (médiane, min 714) depuis 2014, jeudi ou vendredi à 09 h 05 ; volet recherche à partir de **décembre 2019** (10 h 40), pic en 2023.
- Clustering de séries / DTW : non pertinents (une seule série dense).

## 7. Bloc 6 — Texte (`clinical_notes`)

**Méthode.** Vérification UTF-8 (aucun caractère non ASCII) et langue (anglais, ~33 % de mots-outils anglais). Découpage en phrases et **dédoublonnage** (42 phrases dupliquées dans 16 notes) ; repérage des phrases « gabarits » (27 phrases présentes dans ≥ 5 notes). Pré-traitement `nltk` (minuscules, ponctuation, tokenisation, PoS, lemmatisation, stopwords anglais + stopwords du domaine). Bag of words, nuage de mots, n-grams (1 à 3), TF-IDF par note, par `document_type` et par `confidentiality_class`, similarité cosinus + IQR / z modifié (bloc 3), CAH cosinus (bloc 2), LSA et LDA, concordancier.

- **26 notes sur 30 sont des gabarits** (23-31 mots une fois le gabarit retiré). **4 notes rédigées** (CLN-510027, -510025, -510055, -510064) : seules anomalies de similarité (IQR et z modifié < −4,8) et 4 singletons du meilleur clustering (k = 9, silhouette 0,64).
- TF-IDF par classe : les notes `restricted` portent *source*, *camera*, *contractile*, *mismatch*, *public*.
- Topics LDA **instables** (Jaccard 0,25 à 0,54 d'une graine à l'autre) : non utilisés comme indices.
- **Indices** : seconde source cardiaque « pas entièrement en phase » (CLN-510027, PAT-31065) ; discordance caméra / thermique, *appearance control* envisagé (CLN-510055, PAT-31115) ; démarche oscillante + regard en partie indépendant + rythme apical variable (CLN-510064, PAT-31012) ; PAT-31065 relié à l'identité publique **Elias Voss** (CLN-510025).
- **Leurres** : *dual cardiac structures* **rejeté** après repositionnement (CLN-510126, PAT-31099) ; *appearance control* = réglage caméra remis par défaut (CLN-510039). Le bag of words ne voit pas la négation : seule la lecture du contexte les écarte.

## 8. Bloc 7 — Anomalies finales et narratif

### 8.1 Tri final : indices, artefacts, leurres

| Nature | Observation | Où |
|---|---|---|
| **Indice principal** | `MRP-42 specialist` : 8 dirigeants (chef de gouvernement de Kintara, ministre d'Avelora, gouverneur du Tembara, dirigeants de Solvane Energy, Nivren Maritime Logistics, Ardena Capital Union, Belen Agro-industries, ancienne présidente de conseil), 30 administrations, **toutes** par `LIC-VLN-MED-1002`, tous les ~742 jours depuis 2014 | blocs 2-5, `narrative` §2 |
| **Indice principal** | Après la 1re administration, âge biologique ≈ **12 ans sous l'âge réel** (−4,6 avant → −12,4 après ; autres patients −1,0 ; Mann-Whitney p ≈ 1e-12 ; 5/5 patients appariés baissent, Wilcoxon p = 0,0625 = minimum possible à n = 5). PAT-31099 : âge biologique 75 → 69 ans pendant que l'âge réel passe de 78 à 83 | `narrative` §4.1 |
| **Indice** | Volet « recherche » : 20 personnes ordinaires, 1 administration chacune (déc. 2019 – avr. 2024), 7 praticiens, **pas d'effet mesurable** (peu de mesures après) | `narrative` §2, §4.1 |
| **Indice principal** | 109 observations `discordant` chez 35 patients : fréquence apicale ≈ +20 bpm au-dessus du pouls | bloc 3, bloc 1 §5 |
| **Indice** | Les 35 discordants ont aussi plus de balancement latéral, de second pic et de vibration thoraciques, une reconnaissance caméra plus faible (4 tests significatifs après Bonferroni) ; témoins (mobilité, cognition, saturation) identiques ; Isolation Forest sans info cardiaque : 19/35 retrouvés | `narrative` §4.2 |
| **Indice** | 4 notes rédigées décrivant ce phénotype ; PAT-31065 ↔ Elias Voss (président de Meridian BioLogistics d'après `officer_filings`) | bloc 6 |
| **Résultat négatif** | `MRP-42` et phénotype discordant **indépendants** (0/8 `specialist` discordants, Fisher p = 0,78) | `narrative` §4.3 |
| **À croiser** | Dossier de PAT-31065 : *Corin Hallen*, né en 1975 ; Elias Voss né en 1961 : identité d'emprunt ou rapprochement faux | `narrative` §4.4 |
| **Expliqué** | PAT-31099, PAT-31174, PAT-31183 (anomalies multivariées du bloc 3) = trois des 8 patients `specialist` | `narrative` §2 |
| Artefact | Inscriptions au 1er du mois ; âges presque uniformes | bloc 1 |
| Artefact | 22 consultations le 2025-06-20 (suivis plafonnés) | bloc 5 |
| Artefact | 8 notes datées avant leur consultation ; 42 phrases dupliquées ; 26 notes gabarits | blocs 3, 6 |
| Signature de protocole | Durées 180 / 240 min, retards 35 / 40 min, heures 09 h 05 / 10 h 40, jeudi-vendredi | blocs 3, 5 |
| Leurre | CLN-510126 (*dual cardiac structures* rejeté), CLN-510039 (*appearance control* = réglage caméra) | bloc 6 |

### 8.2 Narratif proposé (partie clinique)

Depuis 2014, **8 dirigeants politiques et économiques de cinq pays** reçoivent tous les deux ans, dans les cliniques de l'Asterion Clinical Institute et par un seul médecin (`LIC-VLN-MED-1002`, Dr. Naima Orrel d'après `public_records`), une procédure confidentielle, `MRP-42` (« metabolic resilience »). Après la première administration, leur âge biologique mesuré passe environ **12 ans sous leur âge réel**. À partir de fin 2019, la même procédure est donnée une seule fois à **20 personnes ordinaires** dans un cadre de « recherche », sans effet mesurable.

En parallèle, **35 patients** présentent un phénotype inexpliqué : une seconde source de battements cardiaques, un balancement plus fort, des signaux thoraciques secondaires et des visages que les caméras reconnaissent moins bien. Les notes les plus confidentielles le décrivent avec prudence, puis relient l'un de ces patients (PAT-31065) à **Elias Voss**, président de Meridian BioLogistics. Les deux fils sont indépendants dans les données cliniques.

**Limites** : données synthétiques avec artefacts ; effectifs très faibles (8 patients `specialist`, 4 notes rédigées) ; `bioage` est une estimation au calcul inconnu ; le lien PAT-31065 ↔ Elias Voss repose sur une seule note et contredit la date de naissance du dossier.

### 8.3 À croiser avec le groupe
- **`public_records`** : `LIC-VLN-MED-1002` = Dr. Naima Orrel (directrice de recherche, Asterion, `REG-VLN-01002`) ; Elias Voss (`REG-VLN-01001`, Meridian BioLogistics) et l'alias *Adrian Vosmere* ; nominations des 8 dirigeants.
- **`logistics`** : références de matériel `MTL-YYYY-NNNN` / `MTL-R-YYYY-NNNN` (une par administration `MRP-42`) à rapprocher des expéditions de lots biologiques vers les cliniques ; Nivren Maritime Logistics (PAT-31074).
- **`ecology`** : Belen Agro-industries (PAT-31142) ; laboratoire Threshold Earth (`FAC-AVL-301`, « Eastern Avelora Agricultural Cluster »), où PAT-31115 a été examiné.
- **`health/cardiac`** : enregistrements de PAT-31065 (`irregular` puis `review_required`) et de PAT-31099 (`repeat_recommended`).

### 8.4 Pistes abandonnées
Retards de 59 et 177 jours (PRC-000054, PRC-000071 : cas isolés) ; effet région ou site sur la discordance ; saisonnalité ; corrélation consultations / discordants (tendance commune) ; topics LDA ; règles d'association sur `patient_administration` ; longueur des textes ; délai avant revue des procédures ; queues des scores bornés ; `MRP-42` → phénotype discordant (rejetée) ; asymétrie du regard et équilibre (non confirmés par les mesures).
