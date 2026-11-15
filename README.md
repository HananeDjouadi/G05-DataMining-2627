# G05 — Le narratif caché : projet Data Mining 2026-2027

Projet de groupe du M2 Intelligence Artificielle / M2 Data Science (UE Data Mining).

**Membres** : DJOUADI Hanane, YENNEK Aldjia, LAIB Merieme, COLMEK Esra, EL OUARRAD Mariam, KOUAMÉ Côme.

L'objectif est de reconstituer le narratif caché dans quatre dossiers de données synthétiques : nettoyage, analyse exploratoire, clustering, règles d'association, détection d'anomalies, graphes, analyse spatiale et temporelle, et fouille de texte. Les données sont fictives et créées à des fins pédagogiques.

## Rendu

- **Rapport final** : [`Rapport_G05_DataMining.ipynb`](Rapport_G05_DataMining.ipynb). Il contient la démarche commune, un chapitre par dossier, les croisements entre dossiers, le narratif reconstitué et les limites de la méthode.

## Structure

```
.
├── Rapport_G05_DataMining.ipynb   rapport final
├── requirements.txt               dépendances (environnement conda)
└── data/
    ├── data_dictionary.csv        description des variables de tous les fichiers
    ├── public_records/            dirigeants, nominations, licences, établissements, mémos
    ├── health/                    dossiers cliniques et signaux cardiaques
    ├── ecology/                   suivi des chauves-souris, climat, appareils, rapports de terrain
    └── logistics/                 ERP, achats, réceptions, stocks, expéditions
```

Dans chaque dossier :

- `*.ipynb` : notebooks d'analyse, un par bloc ou par fichier ;
- `*.csv` : données brutes ;
- `cleaned/` : fichiers nettoyés produits par les notebooks ;
- `syntheses/` : synthèses écrites par dossier (anomalies, choix techniques, preuves) ;
- `figures/` ou `images/` : figures extraites des sorties des notebooks et référencées par les synthèses.

## Synthèses par dossier

| Dossier | Synthèse |
|---|---|
| Public records | `data/public_records/synthèses/synthese_public_records.md` |
| Santé | `data/health/syntheses/SYNTHESE_DOSSIER_HEALTH.md`, `data/health/syntheses/synthese_cardiac.md` |
| Écologie | `data/ecology/syntheses/synthese_ecology.md` |
| Logistique | `data/logistics/syntheses/synthese_logistics.md` |

## Reproduire les analyses

1. Créer l'environnement : `conda create --name <envname> --file requirements.txt`
2. Lancer les notebooks depuis le dossier de leur domaine (par exemple `data/ecology/`), avec *Restart & Run All*. Les notebooks lisent leurs CSV relativement à leur dossier.
3. Les fichiers `cleaned/` sont écrits par la cellule de nettoyage de chaque notebook qui les produit (par exemple `data/ecology/bat_surveys.ipynb` écrit `cleaned/bat_surveys_cleaned.csv`). Les notebooks suivants les lisent : il faut donc lancer les notebooks qui écrivent dans `cleaned/` avant les autres. Dans le dossier `health/`, le nettoyage est fait par `cleaning.ipynb`, à lancer en premier.
