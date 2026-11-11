# SYNTHÈSE LOGISTICS — RÉCEPTIONS ET AJUSTEMENTS DE STOCK

**Master 2 Data Science — Groupe 05 | Année 2026–2027**

Cette synthèse résume nos deux notebooks : `goods_receipts_anomalies.ipynb` et `inventory_adjustments_anomalies.ipynb`. Les détails des calculs et les graphiques sont conservés dans les notebooks.

## SYNTHÈSE EXÉCUTIVE

| Aspect | Résultat principal |
| :--- | :--- |
| **Données étudiées** | 610 réceptions de marchandises et 140 ajustements de stock |
| **Réceptions en attente qualité** | 34/610, soit **5,57 %** |
| **Pics de réceptions** | **3 mois** retenus après correction BH, tous à `FCL-1011` |
| **Taux qualité anormal par établissement** | **Aucun** retenu après correction BH |
| **Ajustements atypiques sur les quantités** | **23 ajustements**, dont **7 grandes amplitudes** dans `medical_consumables / case` |
| **Pics mensuels d’ajustements** | Aucun pic retenu après correction BH |
| **Chronologie des références** | **21 ajustements** citent une réception ou expédition datée après l’ajustement |
| **Analyses exploratoires ajoutées** | Pearson/Spearman, nuages de points, ACF, STL et graphe des références |

**Idée principale :** les méthodes nous aident à repérer des périodes, établissements ou ajustements à vérifier. Elles ne permettent pas, à elles seules, d'affirmer qu'il existe une erreur de saisie ou une livraison manquante.

## Sommaire

1. [Goods receipts — réceptions](#1-goods-receipts--réceptions)
2. [Inventory adjustments — mouvements de stock](#2-inventory-adjustments--mouvements-de-stock)
3. [Connexion entre les deux fichiers](#3-connexion-entre-les-deux-fichiers)
4. [Bilan transversal](#4-bilan-transversal)

---

## 1. GOODS RECEIPTS — RÉCEPTIONS

### 1.1 Prétraitement et exploration

Le fichier nettoyé contient **610 réceptions** (612 dans le fichier brut), entre juillet 2014 et juin 2025. Les lignes sont uniques et les dates et quantités ont été contrôlées dans le notebook de nettoyage, qui reste séparé.

- `receipt_status` permet de distinguer les réceptions `posted` et `quality_hold`.
- **34 réceptions** sont en `quality_hold` (**5,57 %**).
- Quatre valeurs d'`inspection_status` ne sont pas renseignées ; elles restent inconnues, sans imputation.
- `FCL-1011` est l'établissement avec le plus de réceptions (**133**), suivi de `FCL-1007` (**76**) et `FCL-1010` (**61**).

**Notre choix :** calculer les taux d'attente qualité avec leur nombre de réceptions. Un taux de 25 % sur seulement 8 réceptions (`FCL-1023` : 2/8) n'a pas la même précision qu'un taux calculé sur plus de 100 réceptions. Les intervalles de Wilson permettent de visualiser cette incertitude.

**Corrélations et graphiques :** les matrices Pearson/Spearman et les nuages de points comparent, sur les 30 établissements, le nombre de réceptions, les mois actifs, les réceptions par mois actif et le taux d'attente qualité. Le nombre de réceptions et les mois actifs sont mécaniquement liés en partie ; les taux de qualité sont fragiles pour les petits établissements.

### 1.2 Bloc 2 — Clustering et réduction de dimension

Les **30 établissements** sont comparés selon leur nombre de réceptions, leurs mois actifs, leur activité moyenne et leur taux de `quality_hold`. Les volumes sont transformés en logarithme puis les variables standardisées.

| Profil K-Means | Établissements | Ce qu’on observe |
| :--- | ---: | :--- |
| Cluster 0 | 16 | Peu de réceptions (médiane **5,5**) |
| Cluster 1 | 9 | Activité plus importante (médiane **37** réceptions) |
| Cluster 2 | 5 | Volume limité, mais attente qualité médiane plus élevée (**14,3 %**) |

K-Means retient **3 groupes**. La CAH donne une partition proche (**ARI = 0,883**). Le dendrogramme, l'ACP et t-SNE permettent de visualiser ces profils.

**Interprétation :** le troisième groupe ne représente pas automatiquement des établissements en difficulté. Certains ont très peu de réceptions : les proportions sont donc instables.

### 1.3 Bloc 3 — Détection d’anomalies

**Fréquences :** les tests IQR/Z signalent **113 mois-établissement distincts**, mais les séries sont très clairsemées et contiennent beaucoup de zéros. Ce nombre ne signifie pas 113 incidents.

**Complément temporel ACF/STL :** une série mensuelle globale couvre **132 mois**, dont **16 sans réception enregistrée**. Une ACF à 24 décalages et une décomposition STL (période annuelle de 12 mois) visualisent la dépendance entre mois, la tendance, la saisonnalité et les résidus. Les zéros désignent une absence de ligne dans cette extraction : on ne sait pas si l'établissement était ouvert ou correctement couvert. **Les résidus STL ne sont donc pas considérés comme des anomalies confirmées.**

Nous avons aussi comparé chaque mois aux 12 mois précédents, avec un test conditionnel binomial puis une correction de Benjamini–Hochberg (BH). Trois pics restent retenus :

| Établissement | Mois | Réceptions | q ajusté BH |
| :--- | :--- | ---: | ---: |
| `FCL-1011` | Mai 2018 | 6 | ≈ 0,000353 |
| `FCL-1011` | Juin 2021 | 9 | ≈ 0,000353 |
| `FCL-1011` | Mars 2022 | 9 | ≈ 0,021735 |

**Qualité :** le test exact de Fisher, suivi de BH, ne retient **aucun établissement** avec un taux d'attente qualité supérieur aux autres au seuil q<0,05. Les taux par trimestre sont visualisés avec les effectifs et une carte thermique.

**Autres méthodes :** Isolation Forest, LOF et DBSCAN comparent les profils des établissements. Elles signalent des profils inhabituels, pas des erreurs confirmées. Avec Apriori, **52 ensembles fréquents et 5 règles** sont trouvés ; les associations concernent notamment l'unité `service` et les volumes élevés, sans interprétation causale.

### 1.4 Vérification avec les commandes

La jointure sur **numéro de commande + ligne de commande** donne :

- **0 réception** dont la commande est absente du fichier de référence ;
- **0 désaccord d'unité** entre commande et réception ;
- **65 lignes de commande** dont la quantité reçue enregistrée est inférieure à la quantité commandée ;
- **0 ligne** dont la quantité reçue dépasse la quantité commandée.

**À vérifier :** les statuts, dates et éventuelles livraisons partielles. Nous n'affirmons pas que les 65 lignes correspondent à des marchandises définitivement manquantes.

---

## 2. INVENTORY ADJUSTMENTS — MOUVEMENTS DE STOCK

### 2.1 Prétraitement et exploration

Le fichier contient **140 ajustements**, dont **89 diminutions** et **51 augmentations**. La quantité signée, l'amplitude absolue, le motif et le statut sont examinés séparément.

**Notre choix :** comparer les quantités **par catégorie de produit et par unité**. Un ajustement négatif signifie une diminution de stock, ce qui est normal. Les références de réception et d'expédition absentes sont optionnelles et ne sont pas automatiquement considérées comme des erreurs.

**Corrélations et graphiques :** les matrices Pearson/Spearman et les nuages de points comparent les établissements sur leurs nombres d'ajustements, leurs mois actifs, la part des diminutions et la part des événements avec référence documentaire. Nous ne corrélons pas directement des quantités exprimées dans des unités différentes. Les proportions d'un petit établissement peuvent être instables.

### 2.2 Bloc 2 — Clustering et réduction de dimension

K-Means retient **5 clusters**. Leur composition suit principalement les cinq catégories d'articles :

| Cluster | Ajustements | Catégorie dominante |
| :--- | ---: | :--- |
| 0 | 14 | `protective_equipment` |
| 1 | 20 | `monitoring_equipment` |
| 2 | 51 | `medical_consumables` |
| 3 | 19 | `agricultural_product` |
| 4 | 36 | `biological_sample` |

La comparaison avec la CAH donne un **ARI de 0,575** : les deux classifications ne sont pas identiques. Les graphiques ACP et t-SNE aident à visualiser les profils. Une autre analyse de projection porte uniquement sur les **51 ajustements `medical_consumables / case`**, afin de ne pas mélanger des unités incompatibles.

**Interprétation :** les catégories utilisées pour construire les profils expliquent une grande partie du clustering. Le regroupement ne permet donc pas de conclure directement qu'un mouvement est anormal.

### 2.3 Bloc 3 — Détection d’anomalies

Les tests IQR, Z-score et Z-score modifié sont appliqués à des groupes comparables. Nous distinguons la quantité signée et l'amplitude absolue.

- **23 ajustements** sont signalés sur les quantités par au moins une comparaison.
- Parmi eux, **7 amplitudes élevées** concernent `medical_consumables / case` ; elles sont affichées avec leurs identifiants dans le notebook.
- Les séries mensuelles produisent **31 mois signalés par IQR/Z**, mais **aucun pic** n'est retenu par le test conditionnel après correction BH.
- `FCL-1010` enregistre **20 ajustements**, sans que ce volume prouve un dysfonctionnement : nous ne disposons pas du dénominateur d'activité correspondant.

**Complément temporel ACF/STL :** la série couvre **132 mois, dont 80 sans ajustement enregistré**. L'ACF et la STL (cycle de 12 mois) servent uniquement à explorer l'évolution des comptes. Les événements sont trop clairsemés pour interpréter les composantes comme des anomalies logistiques certaines : les mois sans ligne ont été codés à zéro dans cette extraction.

Sur le sous-groupe des 51 ajustements médicaux en `case`, Isolation Forest signale **3 cas**, LOF **18** et DBSCAN **6 points isolés**. Ce sont des définitions différentes de l'atypicité, dépendantes des paramètres.

**Associations :** la recherche de paires de modalités donne **6 règles** avec les seuils choisis. L'analyse Apriori complémentaire donne **83 règles retenues**, dont **58** impliquant un motif, un signe ou un statut. Les règles ne prouvent ni intention ni irrégularité.

### 2.4 Vérification des références et de la chronologie

Nous avons comparé les ajustements aux réceptions et expéditions nettoyées :

| Contrôle | Résultat |
| :--- | ---: |
| Références de réception renseignées | 23 |
| Références d'expédition renseignées | 37 |
| Références renseignées introuvables | 0 |
| Réceptions liées datées après l'ajustement | 11 |
| Expéditions liées datées après l'ajustement | 13 |
| **Ajustements distincts avec au moins une référence future** | **21** |

**Interprétation :** ces **21 ajustements** sont de bonnes pistes de vérification documentaire. Cependant, nous ne savons pas encore exactement quel lien métier est représenté par `related_receipt_number` et `related_shipment_number`. On ne peut donc pas qualifier ces dates d'impossibles sans cette information.

**Graphe des références :** les **60 liens documentaires renseignés** forment un graphe de **115 nœuds** (ajustements, réceptions et expéditions) répartis en **55 composantes** : 50 de taille 2 et 5 de taille 3. Les ajustements sans référence ne sont pas représentés. La structure du graphe permet de visualiser les liens connus, mais **n'atteste pas le sens métier ni la chronologie des documents**.

---

## 3. CONNEXION ENTRE LES DEUX FICHIERS

Les deux analyses se complètent :

- `goods_receipts` permet de voir les **réceptions enregistrées**, leurs dates et les attentes qualité ;
- `inventory_adjustments` permet d'observer les **corrections de stock** et leurs références vers les mouvements logistiques.

Les références de réception renseignées dans les ajustements sont toutes retrouvées. Ce rapprochement permet de vérifier la chronologie, mais il faut aussi consulter les motifs et les statuts. Le notebook collectif `analyse_logistics.ipynb` fournit un cadre supplémentaire pour croiser commandes, réceptions, ajustements et expéditions.

**Point à discuter avec le groupe :** `FCL-1029` figure dans les ajustements et le référentiel brut des établissements, mais pas dans le référentiel nettoyé. Il faut comprendre le choix de nettoyage avant de modifier les données ou d'en déduire une erreur.

## 4. BILAN TRANSVERSAL

Les deux fichiers présentent des observations atypiques : pics de réceptions, quantités inhabituelles, profils d'établissements différents et dates de références à examiner. Nous avons préféré **conserver les valeurs extrêmes** et les contextualiser plutôt que les supprimer automatiquement.

**Limites :** les décompositions STL et les ACF portent sur des **comptes d'événements enregistrés**, et non des mesures mensuelles continues de stock ou de qualité. Elles sont exploratoires et ne servent pas à confirmer des anomalies. Les tests de fréquence reposent sur des hypothèses de couverture continue, d'exposition comparable et de stabilité locale. Les fenêtres mensuelles se chevauchent. Les taux qualité reposent parfois sur de très petits effectifs. Les résultats des méthodes de clustering et d'anomalies dépendent des variables et des paramètres choisis.

**Pour continuer :** vérifier en priorité les trois pics de réception, les sept grandes amplitudes d'ajustements et les 21 cas de chronologie surprenante. Leurs identifiants et tableaux détaillés se trouvent dans les notebooks. Les anomalies restent des **pistes**, pas des preuves de fraude ou de livraison manquante.

*Fichiers source nettoyés conservés ; aucun export CSV supplémentaire d'anomalies. Les notebooks de nettoyage et les autres travaux du groupe n'ont pas été modifiés.*
