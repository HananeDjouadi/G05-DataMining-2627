# Rapport public records : anomalies détectées

Neuf fichiers ont été analysés, en deux ensembles :

- **Registres de personnes, licences et documents** : `officer_filings`, `public_appointments`, `licensed_facilities`, `licensed_professionals`, `institutional_memos`, `public_and_investigative_documents`.
  - Notebooks : `exploration.ipynb` (bloc 1), `clustering.ipynb` (bloc 2), `anomalies.ipynb` (bloc 3). « anomalies, cellule 16 » désigne la cellule `In [16]` de `anomalies.ipynb` après *Restart & Run All*.
  - Les figures sont dans `../figures/`, extraites des sorties des notebooks.
- **Registre des entités et journaux d'événements** : `registered_entities`, `regulatory_case_events`, `site_access_case_events`.
  - Notebooks : `clustering_public_records.ipynb` et `anomalies_public_records.ipynb`.

Chaque anomalie est présentée avec sa méthode, les chiffres issus des notebooks et ses preuves.

---

## Résumé

- **8 personnes sont au sommet depuis 39 à 57 ans** (`public_appointments`, `officer_filings`) : 4 dirigeants politiques et 4 chefs d'entreprise. Ce sont les seuls patients à recevoir les 30 procédures `MRP-42` « restricted_specialist » du dossier Health, toutes réalisées par le même médecin d'Asterion.
- **Deux lignées de noms se succèdent depuis 1840 sans jamais se chevaucher** (`officer_filings`) et aboutissent à Elias Voss (Meridian) et Jacob Vale.
  - Les entreprises de ces lignées sont les entités dissoutes du XIXᵉ siècle de `registered_entities`.
  - Les licences des noms anciens sont toujours actives, pour des titulaires qui auraient 125 à 163 ans (`licensed_professionals`).
- **Horizon Field Research Cooperative** a un comportement d'accès unique (`site_access_case_events`) : 18 dossiers, accès de nuit de 18 h à 03 h, revue de sécurité, dossiers confidentiels. À KNT-KES, ses accès tombent quelques jours avant les deux fortes baisses de chauves-souris relevées par le dossier Ecology (avril 2021 et avril 2024). Horizon partage aussi 4 dirigeants avec Ardena Capital Union, dirigée par l'une des 8 élites.
- **Des incohérences de dates** :
  - 9 mandats finissent avant d'avoir commencé, dont 3 qui cachent un double poste entre Asterion et l'administration de Tembara ;
  - un memo est signé en 2018 par un dirigeant dont le mandat a pris fin en 1991 ;
  - Elira Vaun, décédée en 2021, apparaît encore dans des dossiers d'accès en 2024.
- **Le récit textuel tient en 11 documents sur 70** : CHAMELEO, Unwabu, « unités biologiques » manquantes, chauves-souris *Eonycteris spelaea*, transfert « non terrestre ».
- **Des plaintes de la société civile visent Meridian** (4 dossiers sur 10) **et les agences qui autorisent les accès d'Horizon** (6 dossiers sur 10, sur les mêmes sites).
- **Les liens entre fichiers et entre dossiers sont des coïncidences de noms, de dates et d'identifiants.** Ils ne démontrent pas un programme : ils indiquent où regarder.

---

## 1. Démarche commune

- **Nettoyage** (`nettoyage.ipynb`) :
  - pipeline commun : dates en formats mixtes converties, doublons sur la clé primaire supprimés (1, dans `officer_filings`), clés et formats d'identifiants vérifiés (tous conformes), espaces multiples corrigés ;
  - `recorded_birth_year` complété à partir des autres nominations de la même personne (9 → 6 manquants) ;
  - aucune valeur extrême supprimée.
- **Résidus de nettoyage** dans les journaux : casse mixte dans `activity` (11 modalités au lieu de 9) et espaces multiples dans `access_purpose` (9 au lieu de 8), harmonisés avant l'analyse.
- **Contrôles de cohérence des journaux**, tous réussis :
  - pas d'événement antérieur au précédent dans un dossier ;
  - chaînage `previous_status` → `new_status` continu ;
  - pas d'événement hors de la vie d'une entité ;
  - un nom pour un seul numéro d'enregistrement ;
  - agence autorisante conforme au préfixe du site.
- **Variables construites** :
  - registres : dates transformées en durées et en années, avec le 31/12/2025 comme date de référence pour les mandats en cours ;
  - journaux : heure, délai depuis l'événement précédent, délai depuis l'ouverture, nombre d'événements et durée du dossier. Les journaux sont aussi **agrégés par dossier**.
- **Détection** :
  - univarié : IQR, z-score (|z| > 3), z-score modifié (|z| > 3,5) ;
  - multivarié : Isolation Forest, LOF et DBSCAN (`RobustScaler` et 10 graines pour les journaux) ;
  - **règles logiques** sur les dates (fin avant début, auteur hors de son mandat, âge d'un titulaire de licence, personne citée après son décès) ;
  - **croisements entre fichiers et avec les autres dossiers**.
- **Patterns** : Apriori et FP-Growth (`mlxtend`), avec support, confidence, lift et leverage.
- **Clustering** : K-Means (coude et silhouette), classification hiérarchique (Ward, ou Hamming pour les variables nominales), DBSCAN, comparaison par l'ARI ; projections PCA et t-SNE.

---

## 2. Cas central : 8 « élites persistantes » et le traitement `MRP-42`

### 2.1 Ce qui est observé

| Personne | Fonction | Depuis | Années | Patient (Health) | `MRP-42` reçus |
|---|---|---|---|---|---|
| Elira Vaun | présidente du conseil d'État (Avelora) | 1965 | 57 (décès 18/09/2021) | PAT-31099 | 3 (2014–2018) |
| Oskar Nivren | président de Nivren Maritime Logistics | 1973 | 53 | PAT-31074 | 4 (2017–2025) |
| Mirek Talven | gouverneur régional (Tembara) | 1974 | 52 (83 ans) | PAT-31174 | 4 (2015–2021) |
| Celene Varro | directrice générale de Solvane Energy | 1977 | 49 | PAT-31119 | 4 (2018–2024) |
| Leora Kassen | cheffe du gouvernement (Kintara) | 1978 | 48 | PAT-31221 | 5 (2016–2024) |
| Rayan Belen | président de Belen Agro-Industries | 1981 | 45 | PAT-31142 | 3 (2019–2023) |
| Tavian Serat | ministre (Avelora) | 1984 | 42 | PAT-31044 | 4 (2017–2024) |
| Adrienne Solva | directrice d'Ardena Capital Union | 1987 | 39 | PAT-31183 | 3 (2016–2020) |

Sources : anomalies, cellule 15 (ancienneté) ; `health/cleaned/patient_administration` et `clinical_procedures`.

- Le groupe compte **8 personnes pour tout seuil entre 30 et 38 ans** (anomalies, cellule 15). Elias Voss est à la frontière, avec exactement 30 ans.
- Les 30 `MRP-42` « restricted_specialist » (*Targeted metabolic resilience administration*) vont **toutes** à ces 8 patients, et à personne d'autre.
- Elles sont **toutes réalisées par LIC-VLN-MED-1002 (Dr. Naima Orrel, Asterion)** : 16 sur le campus FAC-VLN-101, 14 à la clinique avelorane FAC-AVL-201.
- L'intervalle entre deux procédures est d'environ **742 jours** (2 ans).
- Les 20 autres `MRP-42` (« research_restricted ») concernent 20 patients ordinaires.

### 2.2 Ce qui n'est pas signalé

- En univarié, seule Elira Vaun ressort (durée de 56,7 ans, anomalies, cellule 13). **Un gouverneur de 83 ans en poste depuis 52 ans n'est pas aberrant statistiquement.**
- En multivarié, seuls Vaun et Talven sont signalés par les 3 méthodes (anomalies, cellule 14).
- Le groupe des 8 n'apparaît qu'en **réunissant deux fichiers** (postes publics et dirigeants d'entreprise), puis en **croisant avec Health**.

![Ancienneté au sommet](../figures/bloc3_16_1.png)

*Années au poste des personnes au sommet (anomalies, cellule 16). En rouge, les 8 personnes qui dépassent 30 ans. Elias Voss est à 30.*

### 2.3 Le document associé

`PUB-510045` (Dr. Lysa Quen, Orison Medical Audit Board, 08/07/2022, restreint) décrit **huit individus** qui reçoivent des interventions de santé récurrentes chez les mêmes cliniciens, souvent juste avant de reprendre des fonctions exigeantes, avec un **âge biologique divergent**. Le document précise qu'il **n'établit ni mécanisme biologique, ni contrepartie, ni objectif politique commun**.

### 2.4 Conclusion sur ce cas

- La concordance est forte : 8 personnes repérées par les dates seules, 8 patients qui sont les seuls à recevoir le même traitement réservé, et un document qui parle de 8 individus.
- **Le rôle du traitement n'est pas démontré.** Un indice va dans ce sens : Vaun ne reçoit plus de `MRP-42` après août 2018, alors que le suivant était attendu vers 2020, et elle décède en septembre 2021.
- Les croisements avec Health ont été vérifiés directement dans `health/cleaned/` ; ils restent à reporter dans un notebook.

---

## 3. Second cas : Horizon Field Research Cooperative et le site KNT-KES

### 3.1 Ce qui est observé

**Horizon Field Research Cooperative** (REG-AVL-01013) est un organisme d'Avelora déclaré comme « environmental contractor ». Dans `site_access_case_events` :
- **18 dossiers d'accès**, 2 à 3 par an de 2018 à 2025, sur 8 sites qui reviennent en rotation (AVL-MAR, AVL-VEL, AVL-ORR, KNT-KES, KNT-SEN, KNT-LUM, TMB-NAC, TMB-PEL) ;
- **accès de nuit, de 18 h à 03 h**, alors que les autres organisations accèdent de 17 h à 11 h le lendemain ;
- dossiers de **34 jours**, contre 12 ou 17 pour les autres ;
- **revue de sécurité** dans 12 dossiers sur 18, et dans aucun dossier d'une autre organisation ;
- **14 dossiers sur 18** classés `confidential` ou `restricted` ;
- 7 dossiers sur 18 avec du **matériel de surveillance portable et de stockage frigorifique** (« portable monitoring and cold-storage equipment ») ;
- c'est le **seul demandeur** du fichier d'accès présent dans `registered_entities` ;
- 3 demandes ont été renvoyées pour conditions supplémentaires puis re-soumises (ACC-2021-0010, ACC-2023-0013, ACC-2024-0015), ce qui est unique dans le fichier.

| Dossier Horizon à KNT-KES | Accès (nuit) | Matériel | Dossier Ecology à KNT-KES |
|---|---|---|---|
| ACC-2018-0002 | 22/03/2018 18 h → 23/03 03 h | surveillance et stockage frigorifique | à vérifier |
| ACC-2021-0009 | 25/03/2021 18 h → 26/03 03 h | équipement de protection | **creux d'avril 2021** : 10,6 colonies contre 28,5 en avril 2020 (−62,8 %) |
| ACC-2024-0014 | 21/03/2024 18 h → 22/03 03 h | acoustique et caméra | **baisse d'avril 2024** : `BAT-0000064`, −65,9 % ; rapport d'intrusion `EFR-510088` du 02/04/2024 (matériel coupé, traces de manipulation) |

Source : `site_access_case_events` (dossiers ACC-2018-0002, ACC-2021-0009, ACC-2024-0014) ; synthèse Ecology.

### 3.2 Ce qui n'est pas signalé

- Le **z-score** ne voit ni les heures d'accès (en paliers : 17 h, 18 h, 03 h), ni les 18 dossiers de 34 jours : le groupe est trop gros (18 sur 60) pour paraître « extrême ». **L'IQR signale 72 événements** sur l'heure.
- **Isolation Forest** repère l'ensemble Horizon : 14 des 18 premiers dossiers sont des dossiers Horizon. **LOF**, qui raisonne localement, place surtout en tête des refus d'autres organisations.
- Les **6 refus d'accès** du fichier ne concernent aucun dossier Horizon.

### 3.3 Les liens avec les autres fichiers

- **`officer_filings`** : Horizon partage **4 dirigeants** avec **Ardena Capital Union**, dirigée par Adrienne Solva, l'une des 8 élites (règle Horizon ⇒ Ardena, lift ≈ 9). Ses coordinateurs de terrain (Anika Dren, Jora Vela, Zalia Dalen, Doran Norel, Uma Wessen) sont tous licenciés au laboratoire Threshold Earth (FAC-AVL-301).
- **Personnes nommées dans les dossiers Horizon** : Liora Corda (8 événements, le plus fréquent), Jalen Quorin et Petra Koral (le double poste Asterion / Tembara, section 5) ; Daria Zaven et Ravin Palen (chercheurs cliniques d'Asterion) ; J. Varlan, Jonah Bell et Julien Marot (lignée B, section 4).
- **`regulatory_case_events`** : dans les 6 plaintes de la société civile qui visent une agence, l'agence visée est **celle qui autorise les accès d'Horizon sur le même site** (AVL-MAR / AVL-VEL, KNT-KES / KNT-SEN, TMB-NAC).

### 3.4 Conclusion sur ce cas

- Le comportement d'Horizon est **collectif et régulier**, donc ce n'est pas une erreur de saisie : un seul demandeur, toujours de nuit, sous revue de sécurité et en confidentialité.
- La **coïncidence de dates à KNT-KES** est frappante : à deux reprises, un accès nocturne d'Horizon précède de quelques jours une baisse d'environ 65 % des colonies. **Le lien n'est pas démontré** : la synthèse Ecology rappelle que des baisses comparables existent sur d'autres sites et d'autres années.

---

## 4. Fichier `registered_entities`

### 4.1 Choix techniques

- 24 entités ; 4 variables catégorielles conservées (type, statut, juridiction, adresse), one-hot puis standardisées (28 colonnes).
- **Clustering** (`clustering_public_records.ipynb`) : K-Means, k = 6 retenu par lecture graphique, la silhouette ne dépassant 0,13 pour aucun k entre 2 et 9. Sans l'adresse, la silhouette ne s'améliore pas (0,08 à 0,16).
- **Règles d'association non appliquées** : 3 variables catégorielles pour 24 lignes.
- **Univarié** sur l'année d'enregistrement (IQR, z-score).

### 4.2 Anomalies

**A. Entités du XIXᵉ siècle** : 3 entités dissoutes sont hors des bornes IQR sur l'année d'enregistrement (1 seule au z-score, effet de masque) :
- **Vann Maritime Advisory** (REG-NDH-01015, 1838–1891) ;
- **Varlan Natural History Circle** (REG-KNT-01018, 1883–1920) ;
- **Warden Medical Ventures** (REG-VLN-01016, 1894–1948).

Ce sont les entreprises des **lignées** de `officer_filings`, comme Bell Conservation Survey (1923), Vosmere Biotechnics (1951–1994) et Marot Ecological Policy Office (1969–2006). Aucune n'est citée par les journaux.

**B. Adresse partagée** : **Whitehaven Strategic Holdings** (REG-VLN-01004, « investment and consulting company ») est enregistrée à la même adresse que le siège de **Meridian BioLogistics** (« Meridian Valenne Headquarters »).

**C. Registre partiel** :
- 13 numéros cités par le journal réglementaire et 24 par le journal d'accès (forme `REG-xxx-2xxxx`) sont absents du registre ;
- 15 entités sur 24 ne sont citées par aucun événement.

**D. Clustering** : les 6 clusters reprennent la **juridiction** (12 Valenne, 4 Avelora, 3 + 2 Nordhaven, 2 Kintara, 1 Tembara), qui est aussi le préfixe du numéro d'enregistrement. Pas de groupe naturel.

### 4.3 Preuves

- Années d'enregistrement 1838, 1883 et 1894, hors IQR (`anomalies_public_records.ipynb`).
- `registered_address` de REG-VLN-01001 et de REG-VLN-01004 identiques.

---

## 5. Fichier `officer_filings`

### 5.1 Choix techniques

- **Variables construites** (exploration, cellule 5) :
  - durée du mandat, délai de déclaration, année de nomination, âge à la nomination ;
  - `birth_year_or_age_band` séparée en année exacte et milieu de tranche (`1970–1979` → 1974,5).
- **Manquants** : les 77 `cessation_date` vides sont exactement les 77 mandats actifs (exploration, cellule 8). Ce sont des mandats en cours, non imputés.
- **Clustering** (clustering, cellules 4 à 8) :
  - variables : année, âge, durée jusqu'en 2025, statut + nationalité (one-hot), standardisées ;
  - exclus : `filing_delay_days` (uniforme, aléatoire) et l'année de naissance (r = 0,98 avec l'année de nomination) ;
  - K-Means k = 5 (silhouette 0,442, contre 0,455 à k = 6 ; k = 5 retenu car plus lisible), comparé à Ward ; DBSCAN avec eps = 1,5 et min_samples = 5.
- **Patterns** (anomalies, cellule 5) : une transaction = une personne ; items = les entreprises où elle a siégé + sa nationalité ; support minimal 0,03.
- **Multivarié** (anomalies, cellule 7) : année, âge, durée ; `contamination = 0,05`.

### 5.2 Anomalies

**A. Nominations du XIXᵉ siècle** (anomalies, cellules 6 et 7) : FLG-000001 Edward Vann **1840**, FLG-000005 J. Varlan **1885**, FLG-000002 Emil Warden **1895**, FLG-000006 Jonah Bell **1924**, FLG-000003 Adrian Vosmere **1952**.
- L'IQR et le z-score modifié signalent les 5 ; le z-score classique seulement 4 (il manque 1952 : effet de masque).
- Les 4 premières sont signalées par **les 3 méthodes multivariées**.
- Chacune est le seul dirigeant connu d'une entreprise dissoute (section 4).

**B. Deux lignées sans chevauchement** (anomalies, cellules 9 et 10) :
- **A** : Vann (1840–1888) → Warden (1895–1943) → Vosmere (1952–1991) → **Elias Voss** (FLG-000004, Meridian, depuis 1996). Écarts de 6, 8 et 4 ans.
- **B** : Varlan (1885–1918) → Bell (1924–1963) → Marot (1970–2004) → **Jacob Vale** (FLG-000008, depuis 2008). Écarts de 5, 6 et 3 ans.
- Les initiales se répètent (E. V. → E. W. → A. V. → E. V. ; J. V. → J. B. → J. M. → J. V.). Les intitulés de la lignée A « s'additionnent » : *shipping → medical investor → biotechnology and logistics → BioLogistics*.
- **Limite** : beaucoup de carrières du fichier s'enchaînent sans chevauchement. C'est l'accumulation des indices qui interroge, pas la succession seule.

**C. 9 mandats à durée négative** (anomalies, cellule 8) :
- 6 se terminent la veille de leur début, 3 un an avant (−366 / −367 j).
- Les 3 cas à un an sont des *Regional advisers* d'Asterion : FLG-000050 Liora Corda, FLG-000077 Jalen Quorin, FLG-000131 Petra Koral (section 6).
- **Aucune méthode statistique ne les détecte** : la borne basse de l'IQR est à −2 786 jours.

**D. Dirigeants partagés entre camps opposés** (anomalies, cellule 5) :
- 27 personnes sur 104 ont siégé dans au moins 2 entreprises ; 6 paires d'entreprises partagent exactement 4 dirigeants.
- **Common Ground Climate Forum ⇒ Meridian–Asterion Foundation** : confidence 1, lift 13. Tous les dirigeants passés par la coalition militante sont aussi passés par la fondation du groupe Meridian–Asterion.
- **Horizon Field Research Cooperative ⇒ Ardena Capital Union** (section 3) ; Nivren ⇒ Whitehaven et Helix ⇒ Solvane (lift ≈ 21 avec la nationalité) ; Meridian ⇒ Valenne Industrial Futures Council.

**E. Clustering** (clustering, cellules 7 et 8) :
- les 5 clusters **sont les 5 nationalités** (62 / 27 / 19 / 17 / 10). K-Means et Ward ne diffèrent que sur 4 lignes ;
- DBSCAN classe 22 points en bruit : les nominations anciennes, des carrières très longues (Belen, Nivren, Solva) et des nationalités peu représentées.

### 5.3 Preuves

![Année de nomination](../figures/bloc3_06_1.png)

*Année de nomination (anomalies, cellule 6). Les 5 nominations antérieures à 1960, en rouge, sont hors IQR.*

![Anomalies multivariées officer_filings](../figures/bloc3_07_1.png)

*Projection PCA (anomalies, cellule 7). Les 4 points rouges, signalés par les 3 méthodes, sont les nominations de 1840 à 1924.*

![Deux lignées](../figures/bloc3_10_1.png)

*Frise des deux lignées (anomalies, cellule 10). Les mandats se succèdent sans se chevaucher de 1840 à 2025.*

![Co-occurrences entre entreprises](../figures/bloc3_05_1.png)

*Matrice de co-occurrences des entreprises et nationalités (anomalies, cellule 5).*

![PCA et t-SNE officer_filings](../figures/bloc2_05_1.png)

*PCA et t-SNE (clustering, cellule 5). Les îlots correspondent aux nationalités.*

---

## 6. Fichier `public_appointments`

### 6.1 Choix techniques

- **Variables écartées** : `appointment_basis` (constante : « appointed ») et `official_region` (identique à `appointing_jurisdiction`) (exploration, cellule 11).
- **Manquants** : les 34 fins de mandat vides correspondent aux mandats sans motif de départ, donc **en cours**. Les 6 années de naissance manquantes sont imputées par la médiane **uniquement pour le clustering**.
- **Variable construite** : un **niveau hiérarchique** déduit du titre (1 = agent, 2 = cadre, 3 = sommet de l'État) (exploration, cellule 12).
- **Clustering** (clustering, cellules 9 à 13) : one-hot de la juridiction et de l'institution, standardisation, K-Means k = 4 (silhouette 0,28), comparé à Ward (ARI 0,51). DBSCAN jugé inadapté (17 dimensions pour 62 lignes).
- **Multivarié** (anomalies, cellule 14) : année de début, durée, niveau, âge ; `contamination = 0,08`.
- **Élites persistantes** (anomalies, cellule 15) : postes de niveau 3 + dirigeants d'entreprise actifs dont l'intitulé nomme l'entreprise (`officer_filings`), avec un seuil de plus de 30 ans.

### 6.2 Anomalies

**A. 8 élites persistantes** : voir section 2.

**B. Double poste caché Asterion ↔ Tembara** (anomalies, cellules 17 et 18) : APT-000026 Liora Corda (depuis le 01/01/2013), APT-000038 Jalen Quorin (depuis le 01/01/2012) et APT-000062 Petra Koral (depuis le 01/01/2005) sont **directeurs administratifs régionaux à la Tembara Coastal Administration**, toujours en poste.
- Leur mandat chez Asterion est déclaré terminé **la veille** de leur nomination publique.
- Pourtant, il commence **après** cette nomination (2014, 2013) ou le même jour (2005).
- Ces trois personnes sont aussi nommées dans les dossiers d'accès d'Horizon (section 3).

**C. Durée et âge** (anomalies, cellule 13) :
- une seule durée hors IQR : APT-000006 Elira Vaun, 56,7 ans ;
- deux âges de nomination atypiques : Mirek Talven (48 ans) et Elira Vaun (45 ans).

**D. Multivarié** (anomalies, cellule 14) : Vaun et Talven sont signalés par les 3 méthodes ; APT-000058 Jalen Havel, agent d'accréditation depuis **1983** (43 ans), par 2.

**E. Patterns** (anomalies, cellule 12) : règles mécaniques (juridiction ⇔ pays de l'institution). 12 nominations sur 62 se font en Avelora par des Kintarans ou des Valennans, toutes en `role_transition` : régularité de génération, non interprétée.

### 6.3 Preuves

![Double poste caché](../figures/bloc3_18_1.png)

*Périodes déclarées (anomalies, cellule 18). La fin déclarée chez Asterion (trait rouge) précède le début (flèche) et tombe la veille de la fonction publique (barre grise).*

![Anomalies multivariées public_appointments](../figures/bloc3_14_1.png)

*Projection PCA (anomalies, cellule 14). En rouge, les postes de Vaun et Talven, signalés par les 3 méthodes.*

![Distributions public_appointments](../figures/bloc1_14_1.png)

*Distributions (exploration, cellule 14). Queue de durées supérieures à 40 ans ; titulaires encore en poste jusqu'à 83 ans.*

---

## 7. Fichier `licensed_facilities`

### 7.1 Choix techniques

- **3 lignes** : ni statistiques descriptives, ni clustering, ni détection d'anomalies. Description seulement (exploration, cellule 17).
- `license_end` (vide) et `facility_status` (« active ») sont constantes.
- Les coordonnées sont fictives : seules les distances relatives sont interprétées.

### 7.2 Constats

- **Asterion Clinical Institute** exploite son campus en Valenne (FAC-VLN-101) et une **clinique « executive health » en Avelora** (FAC-AVL-201, ouverte en 2008).
- Cette clinique est à environ 22 km du **laboratoire Threshold Earth** (FAC-AVL-301), dans l'*Eastern Avelora Agricultural Cluster*.
- Le laboratoire, **environnemental**, déclare un service de *non-invasive human assessment*.

---

## 8. Fichier `licensed_professionals`

### 8.1 Choix techniques

- **Variables écartées** : `license_status` (constante : « active ») et `license_expiry_date` (vide). `primary_facility_license` est vide pour les administratifs et les juristes : recodé en « aucun ».
- **Clustering** (clustering, cellules 15 à 19) : 5 variables nominales en one-hot, **distance de Hamming**, classification hiérarchique `average`, k = 4 (silhouette 0,67) ; K-Means identique (ARI 1) ; DBSCAN avec eps = 0,1.
- **Multivarié non réalisé** : une seule variable numérique (l'année de délivrance).
- **Croisement** (anomalies, cellule 22) : années de naissance reprises de `officer_filings`.

### 8.2 Anomalies

**A. Licences anciennes toujours actives** (anomalies, cellules 21 et 22) :
- LIC-KNT-RES-1007 J. Varlan (**1886**), LIC-VLN-MED-1001 Emil Warden (**1895**, cardiologue sur FAC-VLN-101), LIC-AVL-RES-1008 Jonah Bell (**1924**) ;
- l'IQR signale 5 années (avec Vaun 1965 et Marot 1970), le z-score modifié 3, le z-score classique 2 ;
- croisés avec leurs naissances, les titulaires auraient **163, 154 et 125 ans en 2025**, alors que leur âge à la délivrance (24 ans) est normal ;
- ces licences sont rattachées à des établissements ouverts entre 2001 et 2012 ;
- **Contrôle** : la licence d'Elira Vaun (LIC-AVL-ADM-1013), décédée en 2021, est aussi restée active. La non-radiation existe donc dans ce registre.

**B. Licence datée du futur** : LIC-VLN-MED-1006 Farah Havel, délivrée le **01/07/2026**.

**C. Clustering** (clustering, cellule 18) : 4 groupes métier.
- administrateurs (21) ;
- laboratoire Threshold Earth (19), dont 2 chercheurs en **recherche clinique** ;
- médecins d'Asterion (8, tous valennans) ;
- juristes (5).

DBSCAN isole les 2 médecins de la clinique avelorane et J. Varlan.

**D. Patterns** (anomalies, cellule 20) : règles définitionnelles (juriste ⇔ conformité ⇔ Nordhaven, lift 10,6).

### 8.3 Preuves

![Année de délivrance](../figures/bloc3_21_1.png)

*Année de délivrance (anomalies, cellule 21). En rouge, les 5 licences hors IQR (1886, 1895, 1924, 1965, 1970), toutes encore actives.*

![Dendrogramme licensed_professionals](../figures/bloc2_18_1.png)

*Dendrogramme en distance de Hamming (clustering, cellule 18), coupé à k = 4.*

Dans Health, la licence d'Emil Warden (LIC-VLN-MED-1001) sert à **147 consultations** (09/2015 → 06/2025) et à 35 procédures. Celle de Farah Havel sert à des consultations dès février 2014, 12 ans avant sa date de délivrance.

---

## 9. Fichier `regulatory_case_events`

### 9.1 Choix techniques

- 232 événements, **40 dossiers** (2016 à 2025), **agrégés par dossier** : nombre d'événements, nombre d'activités, catégorie, agence et source modales, confidentialité (13 colonnes).
- **Clustering** (`clustering_public_records.ipynb`) : K-Means k = 5, comparé à Ward. La silhouette augmente avec k (0,24 pour k = 2, 0,46 pour k = 5, 0,65 pour k = 14) : chaque catégorie forme déjà un petit groupe homogène, donc k = 5 est un choix de lecture.
- **Patterns** : Apriori et FP-Growth au niveau événement (support ≥ 0,04, confiance ≥ 0,8, lift ≥ 1,5). `assigned_unit` (identique à `initiating_source`) et `outcome` (quasi redondant avec `activity`) sont écartés pour éviter des règles triviales.
- **Anomalies** : IQR et z-score sur les délais ; Isolation Forest et LOF au niveau dossier.

### 9.2 Anomalies

**A. 10 dossiers de la société civile** : 8 plaintes environnementales et 2 inspections d'expédition, ouvertes par du matériel transmis par 5 organisations (Common Ground, Lantern, Earth Ledger, Threshold Earth, Pollinator Justice Network).
- Le cluster des **8 plaintes** est le seul pur à la fois en source, en catégorie et en agence : elles sont **toutes traitées par la Tembara Customs Coordination Unit** (règle à 100 %, lift 3,7). Les 2 inspections le sont par l'**Orison Medical Audit Board** (lift 6,1).
- **Cibles** :
  - **4 visent Meridian BioLogistics** (REG-2020-0003 à TMB-PEL, REG-2021-0005 à AVL-MAR, REG-2023-0008 à KNT-KES, REG-2024-0009 à AVL-MAR) ;
  - les 6 autres visent l'agence qui **autorise les accès d'Horizon** sur le même site (section 3).
- Chaque dossier cite une expédition (SHP-…), à retrouver dans Logistics.
- Ces dossiers ne sont **jamais classés sans suite** (contre 25 dossiers de routine sur 30), et ils sont horodatés à 09:00 (contre 10:00 pour la routine).
- Ils ne sont pas extrêmes en multivarié : 3 dans le top 10 de l'Isolation Forest, soit leur part du fichier.

**B. 5 inspections de faune atypiques** (REG-2018-0017, 2018-0029, 2022-0023, 2022-0035, 2023-0011) :
- seuls dossiers de routine à 7 événements et 35 jours, et seuls à aboutir à `corrective action accepted` ;
- signalés par l'IQR sur les délais et par Isolation Forest et LOF, avec REG-2024-0009.

**C. Agences et sujets** :
- 7 organismes publics sont à la fois agence et sujet ou demandeur. Par exemple, la Tembara Customs Coordination Unit traite les plaintes de la société civile et a elle-même demandé un accès, refusé (ACC-2023-0047) ;
- les audits médicaux et les inspections de faune sont confiés à des agences sans lien évident (marchés publics, douanes, sécurité maritime). Un audit médical vise **Whitehaven Strategic Holdings** (REG-2019-0031) ;
- 40 dossiers seulement : à confirmer.

**D. Clustering** : les 5 clusters redonnent la catégorie (K-Means = Ward, ARI = 1) : 15 dossiers de routine, 8 plaintes de la société civile, 7 inspections d'expédition, 5 inspections de faune, 5 audits médicaux.

### 9.3 Preuves

- Règles d'association à 100 % et clusters (`clustering_public_records.ipynb`, `anomalies_public_records.ipynb`).
- Dossiers REG-2020-0003, 2021-0005, 2023-0008 et 2024-0009 (sujet : REG-VLN-01001, Meridian BioLogistics).

---

## 10. Fichier `site_access_case_events`

### 10.1 Choix techniques

- 412 événements, **60 dossiers** (2015 à 2025), **agrégés par dossier** : nombre d'événements, nombre d'activités, agence, objet, matériel et confidentialité modaux (18 colonnes).
- **Clustering** : K-Means k = 5 (silhouette 0,17, croissante avec k), comparé à Ward (ARI = 0,5 : clustering peu stable).
- **Patterns** : au niveau dossier (support ≥ 0,10) et au niveau événement (support ≥ 0,04).
- **Anomalies** : IQR et z-score sur l'heure et la durée ; Isolation Forest (10 graines) et LOF au niveau dossier, **sans l'identité de l'organisation**, pour voir si le comportement suffit à isoler un demandeur.

### 10.2 Anomalies

**A. Horizon Field Research Cooperative** : voir section 3.
- **15 des 18 dossiers Horizon** forment un même cluster.
- Règles : `security_review` ⇒ accès de nuit (100 %, 12 dossiers) ; accès de nuit = dossier de 34 jours = demandeur présent dans le registre (mêmes 18 dossiers) ; au niveau événement, `restricted` + matériel acoustique ⇒ matériel frigorifique (25 événements, lift 7,4).

**B. Demandes renvoyées puis re-soumises** : ACC-2023-0013, ACC-2024-0015 et ACC-2021-0010, toutes d'Horizon, sont les 3 premiers dossiers de l'Isolation Forest.

**C. Refus** : 6 `access denied`, dont ACC-2018-0019, 2018-0040, **2022-0033** et 2023-0026, isolés par LOF. Aucun ne concerne Horizon.
- ACC-2022-0033 est la demande de **Kestrel Conservation Services** sur la grotte **KNT-UJJ** (octobre 2022, refusée).
- Le dossier « Kestrel » est cité par le lanceur d'alerte (PUB-510079).

**D. Personnes citées après leur décès ou depuis très longtemps** (croisement avec les registres de personnes) :
- **Elira Vaun**, décédée le 18/09/2021, est nommée dans des événements d'accès en **mai, juillet et août 2024** ;
- **J. Varlan, Jonah Bell et Julien Marot** (lignée B) sont nommés dans des dossiers de 2022 à 2025.

**E. Clustering** : en dehors du cluster Horizon, les 4 autres clusters sont regroupés par objet : pollinisation de vergers (14), maintenance de capteurs (12), évaluation de restauration d'habitat (10), récupération de matériel d'urgence (9).

### 10.3 Preuves

- Heures d'accès d'Horizon (18 h → 03 h) contre 17 h → 11 h pour les autres ; 72 événements hors IQR sur l'heure, aucun au z-score (`anomalies_public_records.ipynb`).
- Dossiers ACC-2018-0002, ACC-2021-0009 et ACC-2024-0014 (Horizon, KNT-KES, mars, de nuit) ; ACC-2022-0033 (Kestrel, KNT-UJJ).

---

## 11. Fichiers `institutional_memos` et `public_and_investigative_documents`

### 11.1 Choix techniques

- Les deux fichiers sont traités **ensemble** : ils ont la même structure, et la mesure de gabarit se calcule sur tout le corpus. Le texte lui-même relève du bloc 6.
- **Mesures construites** (anomalies, cellule 23) :
  - `n_words` : longueur ;
  - `boilerplate_share` : part des paragraphes présents à l'identique dans au moins 3 documents ;
  - `n_refs` : nombre de références `PO-`, `SHP-`, `ADJ-`, `ACC-`, `REG-`.
- **Clustering non réalisé** : il ne reste que 2 variables nominales par fichier (clustering, cellule 20).
- **Règle logique** (anomalies, cellule 27) : l'auteur appartient-il à l'organisation à la date du document ? Vérifié avec les mandats de `officer_filings` et de `public_appointments`.

### 11.2 Anomalies

**A. 11 documents sans aucun texte gabarit** (anomalies, cellule 25) :
- le z-score modifié les isole. L'**IQR échoue** : sa borne basse est négative, donc inatteignable pour une proportion ;
- le multivarié confirme : les 4 documents signalés par au moins 2 méthodes en font tous partie (anomalies, cellule 26) ;
- les 59 autres documents sont des variations d'un même gabarit, et 27 répètent mot pour mot certains de leurs paragraphes.

| Document | Auteur | Contenu |
|---|---|---|
| `MEM-510100` | Soren Kade, Meridian | Instruction **CHAMELEO** (envoi SHP-2022-00011) |
| `MEM-510101` | Soren Kade, Meridian | Autorité **Unwabu**, escalade vers le bureau exécutif de Meridian |
| `MEM-510062` | Soren Kade, Meridian | « Unités biologiques » manquantes (ADJ-2022-00088) |
| `PUB-510045` | Dr. Lysa Quen | 8 dirigeants, interventions récurrentes (section 2) |
| `PUB-510079` | Mara Venn | *Eonycteris spelaea* pour « consommation », dossier « Kestrel » |
| `PUB-510063` | Mara Venn | Classification « Chamaeleonidian », transfert « non terrestre » |
| `PUB-510044` | Jora Menne | Continuité d'identité de Jacob Vale |
| `PUB-510076`, `PUB-510043`, `PUB-510033` | archives | Alias, Wren Street, « make the conditions ordinary » (1942, 1962) |
| `PUB-510094` | Selene Aras | Stratégies écologiques opposées |

**B. Memo signé par un dirigeant hors mandat** (anomalies, cellule 27) :
- **MEM-510128** (15/05/2018) est signé **Adrian Vosmere** pour Vosmere Biotechnics ;
- or son mandat a pris fin en **1991** et l'entreprise a été **dissoute en 1994** (`registered_entities`), et le memo porte sur l'envoi SHP-2020-00019, daté de 2020 ;
- au total, 6 documents sont signés hors mandat. Les autres sont plausibles : anciens employés, déclaration tardive.

**C. Leurres** : des paragraphes uniques ajoutés à des documents gabarits donnent une explication banale des termes clés.
- CHAMELEO serait un ancien tableau de bord (MEM-510089) ;
- Unwabu, un bateau ou une référence folklorique (PUB-510127, 1995) ;
- « non-terrestrial », des données satellitaires (PUB-510121) ;
- « external destination », un client hors réseau (MEM-510006, signé hors mandat).

**D. Patterns** (anomalies, cellule 24) : règles définitionnelles (type ⇒ collection) ; règles faibles sur les memos (support 0,067).

### 11.3 Preuves

![Part de texte gabarit](../figures/bloc3_25_2.png)

*Part de texte gabarit (anomalies, cellule 25). Les 11 documents à 0 sont à gauche ; aucun n'est hors IQR.*

![Anomalies multivariées documents](../figures/bloc3_26_1.png)

*Projection PCA (anomalies, cellule 26). Les 4 documents signalés par au moins 2 méthodes font partie des 11 documents signal ; MEM-510100 (CHAMELEO) est signalé par les 3.*

---

## 12. Croisement des anomalies

```
8 élites persistantes (public_appointments + officer_filings)
   │
   ├── Health : seuls patients des 30 MRP-42 restricted_specialist, tous faits par LIC-VLN-MED-1002 (Naima Orrel)
   ├── PUB-510045 : « eight individuals », interventions récurrentes, âge biologique divergent
   ├── Talven ← double poste caché Asterion ↔ Tembara Coastal Administration (Corda, Quorin, Koral)
   └── Solva (Ardena Capital Union) ← 4 dirigeants communs avec Horizon

Horizon Field Research Cooperative (site_access_case_events)
   │
   ├── accès de nuit 18 h → 03 h, revue de sécurité, 14/18 dossiers confidentiels, matériel frigorifique
   ├── KNT-KES : accès les 25/03/2021 et 21/03/2024 → Ecology : baisses d'avril 2021 (−62,8 %) et d'avril 2024 (−65,9 %, intrusion EFR-510088)
   ├── personnes nommées : Corda, Quorin, Koral (Asterion) ; Zaven, Palen (cliniciens) ; Varlan, Bell, Marot (lignée B)
   └── regulatory_case_events : 6 plaintes de la société civile visent les agences qui l'autorisent

Lignées Vann → Warden → Vosmere → Voss  et  Varlan → Bell → Marot → Vale (officer_filings)
   │
   ├── registered_entities : leurs entreprises sont les entités dissoutes de 1838, 1883, 1894…
   ├── licensed_professionals : licences de 1886, 1895, 1924 toujours actives
   ├── Health : licence de Warden utilisée pour 147 consultations, 2015–2025
   ├── MEM-510128 : Vosmere signe un memo en 2018
   └── archives : alias, Wren Street, « Chamaeleonidian »

Programme CHAMELEO (memos Meridian)
   │
   ├── Logistics : SHP-2022-00011 déclaré « protective_equipment, ordinary » ; SHP-2023-00035 « legal_service »
   ├── regulatory_case_events : 4 plaintes de la société civile contre Meridian, chacune liée à une expédition
   ├── registered_entities : Whitehaven à la même adresse que Meridian
   └── PUB-510079 : chauves-souris Eonycteris spelaea pour « consommation », dossier « Kestrel » (ACC-2022-0033, KNT-UJJ, refusé)
```

| Fichier | Anomalies | Lien avec les autres fichiers ou dossiers |
|---|---|---|
| `registered_entities` | Entités de 1838, 1883, 1894 ; Whitehaven à l'adresse de Meridian ; registre partiel | `officer_filings` (lignées), memos (Vosmere dissoute) |
| `officer_filings` | 5 nominations de 1840 à 1952 ; 2 lignées ; 9 durées négatives ; ponts entre camps opposés | `licensed_professionals` (mêmes noms), `public_appointments` (double poste), Horizon ↔ Ardena |
| `public_appointments` | 8 élites ; double poste caché ; Jalen Havel depuis 43 ans | Health (`MRP-42`), PUB-510045, dossiers d'accès Horizon |
| `licensed_facilities` | Laboratoire environnemental qui examine des humains | `licensed_professionals` (chercheurs cliniques), Ecology (zone des grottes) |
| `licensed_professionals` | Licences de 1886 à 1924 actives ; licence datée de 2026 | `officer_filings` (naissances), Health (activité réelle) |
| `regulatory_case_events` | 10 dossiers de la société civile (4 contre Meridian) ; 5 inspections de faune atypiques | Logistics (expéditions), Horizon (agences visées) |
| `site_access_case_events` | Horizon : nuit, sécurité, confidentialité ; Vaun citée en 2024 ; refus Kestrel | Ecology (KNT-KES), `officer_filings` (Ardena), registres de personnes |
| `institutional_memos` | 3 memos signal ; MEM-510128 hors mandat | Logistics (SHP, ADJ), `officer_filings` (lignée A) |
| `public_and_investigative_documents` | 8 documents signal ; leurres | Health (PUB-510045), Ecology (PUB-510079), lignées (archives) |

---

## 13. Conclusion

1. **Les 8 élites persistantes sont l'indice le plus solide** : elles sont repérées par les dates, isolées dans Health, seules à recevoir le traitement `MRP-42` réservé, et nommées par un document. **Le rôle du traitement n'est pas démontré** : il reste à vérifier leur âge biologique et le lien entre l'arrêt du traitement et le décès d'Elira Vaun.
2. **Horizon Field Research Cooperative est le seul comportement collectif atypique des journaux** : accès de nuit sous revue de sécurité, dossiers confidentiels, matériel frigorifique. Ses accès à KNT-KES précèdent de quelques jours les deux plus fortes baisses de chauves-souris du site. **Le lien de cause n'est pas démontré.**
3. **Les deux lignées, les entités du XIXᵉ siècle et les licences de 125 à 163 ans** suggèrent une continuité d'identité qui aboutit à Elias Voss et Jacob Vale.
   - **L'explication administrative (licences non radiées) existe** dans le registre.
   - Elle n'explique ni l'activité réelle de la licence de Warden entre 2015 et 2025, ni la présence de noms anciens ou de personnes décédées dans des dossiers récents.
4. **Les incohérences de dates** (durées négatives, memo de Vosmere, Vaun citée en 2024) ne sont visibles que par des règles logiques : aucune méthode statistique ne les détecte.
5. **Les clusters ne révèlent rien de caché** dans la plupart des fichiers : ils retrouvent la nationalité, la juridiction, le métier ou la catégorie. **Seuls deux groupes ont un sens** : le cluster Horizon et le cluster des plaintes de la société civile.
6. **Pour confirmer**, il faut :
   - l'âge biologique et les observations cardiaques des 8 élites, de Voss et de Vale (Health) ;
   - le contenu réel des envois CHAMELEO, de l'ajustement ADJ-2022-00088 et des expéditions citées par les plaintes contre Meridian (Logistics) ;
   - les colonies d'*Eonycteris* à KNT-KES en mars-avril 2018, et sur les autres sites visités par Horizon, aux mêmes dates (Ecology) ;
   - un graphe personnes × organisations (bloc 4).

---

## 14. Index des figures

| Figure | Source (cellule) | Ce qu'elle montre |
|---|---|---|
| `bloc3_16_1.png` | anomalies (16) | Ancienneté au sommet, 8 personnes au-delà de 30 ans |
| `bloc3_06_1.png` | anomalies (6) | Année de nomination, 5 nominations hors IQR |
| `bloc3_07_1.png` | anomalies (7) | Anomalies multivariées de `officer_filings` |
| `bloc3_10_1.png` | anomalies (10) | Frise des deux lignées 1840–2025 |
| `bloc3_05_1.png` | anomalies (5) | Co-occurrences entreprises × nationalités |
| `bloc2_05_1.png` | clustering (5) | PCA et t-SNE de `officer_filings` |
| `bloc3_18_1.png` | anomalies (18) | Double poste caché Asterion ↔ Tembara |
| `bloc3_14_1.png` | anomalies (14) | Anomalies multivariées de `public_appointments` |
| `bloc1_14_1.png` | exploration (14) | Distributions de `public_appointments` |
| `bloc3_21_1.png` | anomalies (21) | Année de délivrance des licences |
| `bloc2_18_1.png` | clustering (18) | Dendrogramme de `licensed_professionals` |
| `bloc3_25_2.png` | anomalies (25) | Part de texte gabarit des documents |
| `bloc3_26_1.png` | anomalies (26) | Anomalies multivariées des documents |
