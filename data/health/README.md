# Dossier Health — partie clinique

Tables : `clinical_encounters`, `clinical_notes`, `clinical_observations`, `clinical_procedures`, `patient_administration`.
Les fichiers `cardiac_*` sont traités à part (`clustering_cardiac.ipynb`, `anomalies_cardiac.ipynb`, `syntheses/synthese_cardiac.md`).

## Ordre d'exécution
Lancer chaque notebook depuis ce dossier (`data/health/`), avec *Restart & Run All* :

| Ordre | Notebook | Contenu |
|---|---|---|
| 0 | `cleaning.ipynb` | Nettoyage → `cleaned/*_cleaned.csv` |
| 1 | `eda.ipynb` | Bloc 1 : typage, statistiques, lois, corrélations |
| 2 | `clustering.ipynb` | Bloc 2 : PCA, t-SNE, CAH (Hamming), K-Means, DBSCAN |
| 3 | `patterns_anomalies.ipynb` | Bloc 3 : Apriori, anomalies univariées et multivariées |
| 4 | `graphs_process.ipynb` | Bloc 4 : graphes patients-professionnels, centralités, communautés, process mining |
| 5 | `spatio_temporal.ipynb` | Bloc 5 : carte des sites, régions, séries, ACF, STL, fenêtre glissante |
| 6 | `text_mining.ipynb` | Bloc 6 : notes cliniques (dédoublonnage, nltk, TF-IDF, n-grams, LSA/LDA) |
| 7 | `narrative.ipynb` | Bloc 7 : tableau d'indices, chronologie, tests d'hypothèses, narratif, usage de l'IA |

Résumé de tous les résultats : `syntheses/SYNTHESE_DOSSIER_HEALTH.md`.

`spatio_temporal.ipynb` et `narrative.ipynb` lisent aussi `../public_records/` (coordonnées des établissements, `officer_filings`).
`text_mining.ipynb` télécharge les ressources `nltk` au premier lancement ; sans internet, il se replie sur des outils simples et le signale.
`pm4py` et `cdlib`/`infomap` sont optionnels : les cellules qui les utilisent s'ignorent proprement s'ils manquent.
