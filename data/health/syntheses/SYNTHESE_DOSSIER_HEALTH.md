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

## 3. Pistes pour le bloc 3 (à vérifier)

- Les **50 procédures `MRP-42`** (28 patients) correspondent exactement aux consultations `specialist_procedure` (30) et `research_procedure` (20), toutes `restricted`.
- Les **109 observations `discordant`** portent toutes sur la fréquence **apicale** (35 patients). Deux notes d'imagerie évoquent une « seconde structure cardiaque » : PAT-31065 fait partie des patients discordants ; PAT-31099 (dossier `deceased`) a reçu la procédure `MRP-42`.
- **2 procédures** réalisées 59 et 177 jours après l'heure planifiée (`PRC-000054`, `PRC-000071`).