# Synthèse dossier Health, fichiers cardiac

*Tables : `cardiac_recordings` (44 enregistrements), `cardiac_signal_samples` (96 925 échantillons, 2 canaux)*

---

## 1. Données

- **Structure** : 1 ligne par enregistrement dans `cardiac_recordings` (métadonnées, contexte, appareil, interprétation automatique, `device_quality`), 1 ligne par échantillon dans `cardiac_signal_samples`, reliée par `recording_id`. 42 patients, enregistrements de 2017 à 2025.
- **Contrôles de cohérence** : les 8 contrôles structurels passent (taille des signaux = `sample_count`, pas de temps régulier, pas d'orphelin, pas de date future, `device_quality` dans [0, 100]).

## Clustering

**Méthode.** Identifiants et colonnes constantes exclus. `cardiac_recordings` : 4 variables numériques standardisées + 4 catégorielles en one-hot (21 colonnes). `cardiac_signal_samples` : agrégation par enregistrement (moyenne, écart-type, min, max de chaque canal, durée max, nombre d'échantillons), puis standardisation. K-Means (coude, silhouette) pour choisir k, classification hiérarchique (Ward, distance euclidienne), visualisation t-SNE.

| Table | k : tailles | Lecture | Silhouette |
|---|---|---|---|
| `cardiac_recordings` (44) | 4 : 17 / 15 / 9 / 3 | 17 enregistrements à 125 Hz, 15 à 100 Hz, 9 enregistrements longs (= ~29 s, dont 7 au repos), 3 enregistrements `rest` de qualité faible (CRD-410001, -410004, -410031 : 76,9 à 85,8) | 0,22 |
| `cardiac_signal_samples` agrégé (44) | 5 : 16 / 13 / 6 / 5 / 4 | 16 enregistrements courts (= ~15 s), 6 longs (= ~29 s), 13 intermédiaires, 5 enregistrements d'effort (`post_exercise`, `stress_assessment`), aucun `normal`, 4 `technical_repeat` à forte variance du canal A | 0,31 |

- Pas de groupes naturels dans `cardiac_recordings` : la silhouette K-Means est plate (0,20 à 0,22 pour k = 2 à 5), k = 4 l'emporte de peu. Les clusters reflètent le protocole d'acquisition : le cluster 0 est à 125 Hz (17/17), le cluster 2 à 100 Hz (15/15). Comme `sample_count` = fréquence \* durée, la durée pèse deux fois dans la distance.
- `device_quality` ne renseigne pas sur le signal : le cluster 2 a la meilleure qualité moyenne (94,9) mais contient 3 des 5 `artefact`. La lecture "meilleures conditions de mesure" est donc à nuancer. Même chose pour le cluster 3 (" sessions faites à la hâte") : on sait seulement qu'il regroupe 3 enregistrements au repos de qualité faible, dont 2 `complex`.
- Sur les signaux, la séparation courts / longs ne fait que redonner la durée (`time_ms_max` et `time_ms_count` en portent chacun l'information). Les groupes informatifs sont les deux plus petits :
  - **4 enregistrements `technical_repeat`** (CRD-410002, -410017, -410028, -410041) : écart-type du canal A de 0,65 (contre = ~0,45), pics proches de 3, corrélation entre canaux A et B de 0,17 à 0,47.
  - **5 enregistrements d'effort** (CRD-410008, -410010, -410020, -410036, -410044) : fréquence cardiaque estimée = ~54 bpm (contre 66 à 74 ailleurs), moyenne du canal B négative. Les deux points isolés du t-SNE sont CRD-410010 et CRD-410020 (d'après une reproduction du t-SNE).

## Patterns et anomalies

Règles d'association (Apriori et FP-Growth, `mlxtend`, support >= 0,10, soit 5 enregistrements) sur les variables catégorielles. Anomalies univariées (IQR, z-score) sur les métadonnées, sur les échantillons bruts et sur 16 statistiques par enregistrement. Anomalies multivariées (Isolation Forest, LOF) au niveau enregistrement, Isolation Forest aussi au niveau échantillon.

| Table | Patterns | Anomalies | Nature |
|---|---|---|---|
| `cardiac_recordings` | 74 itemsets, `artefact` -> `baseline_variation` (5/5, lift 4,9), `post_exercise` -> `inconclusive` (5/7, lift 2,4), `quality_band=low` -> `rest` (5/5, lift 2,2) | Aucun outlier univarié franc : seul `device_quality` de CRD-410001 (76,9, z = −2,85) est limite. Multivarié : 5 signalés par Isolation Forest et par LOF, 3 communs (CRD-410029, -410010, -410020) | Cas limites, le noyau multivarié est à confronter aux étiquettes |
| `cardiac_signal_samples` | Non applicable (variables numériques) | IQR et z-score signalent ≈ 6,6 % et 2,6 % des échantillons : ce sont les **pics R**. 42 enregistrements sur 44 ont au moins un |z| > 4 intra-enregistrement. Isolation Forest échantillon : CRD-410029 en tête (4,5 % de ses échantillons) | Forme normale du signal, pas des erreurs |
| Les deux tables | _| 2 paires patient + date (PAT-31162, PAT-31065), chacune `rest` puis `technical_repeat` | Attendu, mais la répétition ne lève pas le doute |

- Sur le signal brut, l'IQR et le z-score ne servent pas : il faut résumer chaque enregistrement (amplitude, bruit, fréquence cardiaque, corrélation A/B) avant de chercher des anomalies.
- CRD-410010 et CRD-410029 sont signalés à chaque exécution (15/15), CRD-410020 13 fois, CRD-410026 et CRD-410002 11 fois. CRD-410010 et CRD-410020 sont déjà `artefact` : cohérent. CRD-410029 est `normal` (`clinical_follow_up`, note `baseline_variation`) et ressort sur toutes les vues : c'est le cas le plus intéressant.
- Isolation Forest recoupe bien les étiquettes (3 des 5 `artefact` signalés) mais ne voit aucun des 14 enregistrements `complex`, `irregular` ou `review_required` : les variables décrivent l'amplitude et le bruit, pas le rythme.
- `artefact` -> `baseline_variation` à 100 % : deux colonnes issues du même traitement automatique, la règle semble décrire le codage des étiquettes plutôt qu'un fait clinique. `post_exercise` -> `inconclusive` semble normal (le mouvement dégrade le signal).
- Avec 44 lignes, des supports de 5 enregistrements et des dizaines de règles testées, tous ces résultats sont des pistes.

## Pistes

- Les 4 `technical_repeat` sans enregistrement d'origine (CRD-410002, -410017, -410028, -410041) sont exactement le cluster signal 4, avec les corrélations A/B les plus faibles du fichier. Les 2 répétitions qui ont un original (PAT-31162, PAT-31065) ont été refaites sur un autre appareil (HelixCard Duo, 30 s) et n'ont rien résolu : `complex` -> `irregular` et `irregular` -> `review_required`.
- **Lien avec la partie clinique** : 3 des 4 patients singuliers de la synthèse clinique ont un enregistrement ici (environ 1 patient sur 6 en a un : 42 sur 240 dossiers) :
  - PAT-31065 (observations `discordant`, note d'imagerie "seconde structure cardiaque") : deux ECG le même jour, rattachés à la même consultation (ENC-000038), `irregular` puis `review_required`.
  - PAT-31099 (dossier `deceased`, procédure `MRP-42`) : CRD-410018 le 2018-09-17, `normal` mais `repeat_recommended`, qualité 83,6, dans le seul établissement différent (FAC-AVL-201).
  - PAT-31183 (dossier ancien) : CRD-410040, `irregular`.
  - PAT-31174 n'a pas d'enregistrement. À confronter avec les dates de décès et de dossier de `patient_administration`.
- **Fréquence cardiaque et contexte** : moyenne estimée de 70,8 bpm au repos contre 64,9 après effort, alors que l'inverse est attendu, le cluster d'effort est à = ~54 bpm (CRD-410020 à 44, CRD-410036 à 45). La détection de pics est approximative (seuil moyenne + 2,5 d'écart type).
- **CRD-410029** : `normal` mais atypique sur toutes les méthodes (= ~51 bpm, relu par LIC-VLN-MED-1006).
- Les deux plus faibles `device_quality` (CRD-410001 à 76,9 et CRD-410031 à 80,9) ont été relus par des licences `LIC-VLN-RES-*` (1044 et 1016, 9 relectures sur 44), hypothèse à tester dans `licensed_professionals`.
- **`device_serial_number`** : 44 numéros distincts pour 44 enregistrements, soit un appareil utilisé une seule fois.
- **Jointures** : `reviewing_license_number` (validité de la licence à la date d'enregistrement), `facility_license_number`, `patient_number` et `encounter_number` (retrouver les originaux manquants des `technical_repeat`).
- **Pistes abandonnées** : IQR / z-score sur les échantillons bruts (pics R), segmentation des enregistrements par durée ou fréquence d'échantillonnage, règles d'association sur effectifs de 5 lignes sans test de robustesse.

