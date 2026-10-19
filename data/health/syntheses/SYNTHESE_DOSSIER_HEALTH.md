# Synthèse — Dossier Health, partie clinique

*Groupe G05 · Tables : `clinical_encounters`, `clinical_notes`, `clinical_observation`, `clinical_procedures`, `patient_administration`*

---

## 1. Nettoyage

- **Doublons exacts** : 6 supprimés (encounters 2, observation 2, procedures 1, patient_administration 1).
- **Dates** : 11 colonnes converties en `datetime` (formats mixtes). Aucune incohérence : pas de fin avant le début, aucune observation hors de sa consultation.
- **Types** : `numeric_value` (observation) contenait ~340 valeurs textuelles (virgule décimale, unité collée, ex. « 82.2 index »), extraites puis converties en `float`.
- **Catégories** : minuscules et espaces nettoyés (ex. `encounter_type` 15 → 10 modalités, `interpretation_code` 9 → 4). Identifiants, licences et codes de procédure laissés intacts.
- **Manquants** : colonne vide `superseded_document_id` supprimée ; absences codées explicitement (`no_device`, `no_material`, `unspecified`, `non_applicable`) ; dates manquantes laissées en `NaT`.
- **Aberrants** : aucune ligne supprimée, pour ne pas perdre d'indices. Sorties : `cleaned/*_cleaned.csv`.

## 2. Bloc 2 — Clustering

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

## 3. Bloc 3 — Patterns et anomalies

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

## 4. Pistes pour le bloc 7 (à vérifier)

- Les **50 procédures `MRP-42`** (28 patients) correspondent exactement aux consultations `specialist_procedure` (30) et `research_procedure` (20), toutes restreintes, de durée et de retard fixes, avec des issues qu'aucune autre procédure n'utilise.
- Les **109 observations `discordant`** portent toutes sur la fréquence **apicale** (35 patients). Deux notes d'imagerie évoquent une « seconde structure cardiaque » : PAT-31065 fait partie des patients discordants ; PAT-31099 (dossier `deceased`) a reçu la procédure `MRP-42`.
- **2 procédures** réalisées 59 et 177 jours après l'heure planifiée (`PRC-000054`, `PRC-000071`).
- **8 notes** antérieures à leur consultation : à traiter comme artefact de génération avant toute inférence.
- **Pistes abandonnées** : règles d'association sur `patient_administration` ; longueur des textes ; délai avant revue des procédures ; queues des scores bornés (`lefteye`, `idcons`...).