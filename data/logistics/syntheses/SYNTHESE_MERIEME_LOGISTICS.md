# Contribution Merieme — anomalies des réceptions et ajustements

État vérifié le 25 octobre 2026. Complément autonome ; la synthèse collective et le notebook local `analyse_logistics.ipynb` sont préservés.

## Organisation et méthode

- `../goods_receipts_anomalies.ipynb` : 610 réceptions ; 4 figures enregistrées.
- `../inventory_adjustments_anomalies.ipynb` : 140 ajustements ; 3 figures enregistrées.
- Sources : versions `_cleaned.csv` de `../cleaned/`. Aucun nouveau nettoyage ni jointure externe.
- Les notebooks existants de nettoyage sont conservés. Les résultats (anomalies, seuils et tableau trimestriel) sont affichés directement dans les notebooks, sans création de CSV de résultats.

Les notebooks `inventory_month_end`, `purchase_order_lines`, `shipments` et `health/patterns_anomalies` emploient notamment IQR, Z-score, Z-score modifié, IF/LOF et, pour la santé, DBSCAN. Les nouveaux notebooks reprennent la structure descriptive, les tableaux croisés, les graphiques et les méthodes statistiques adaptées.

Les identifiants ne deviennent pas des mesures numériques. Les quantités de catégories/unités différentes ne sont jamais additionnées dans un indicateur global.

### Compléments justifiés pour les effectifs

Les quantités continues ne suivent pas le même mécanisme que des comptes mensuels souvent nuls ou des proportions à petits dénominateurs. En complément des seuils IQR/Z du groupe :

- **Pic mensuel** : comparaison aux 12 mois précédents. Sous hypothèse de taux constant et d'expositions mensuelles égales, conditionnellement au total des 13 mois, le compte courant suit Binomiale(total, 1/13). Test unilatéral, puis Benjamini–Hochberg (BH), q<0,05, sur toutes les fenêtres évaluables du fichier.
- **Taux qualité par établissement** : test exact de Fisher, unilatéral, contre les autres établissements ; BH sur tous les établissements. Les intervalles de Wilson à 95 % montrent l'incertitude des proportions.
- Les mois sans événement entre première et dernière observation valent zéro **sous hypothèse de couverture continue**. Les mois aux bords des historiques sont exclus des tests, ainsi que les fenêtres dont l'historique n'a pas 12 mois internes complets.
- Les seuils IQR/MAD nuls et les groupes trop petits sont explicitement non évaluables ; pas de division par zéro ni de quota forcé.

Ces compléments ne prouvent pas une causalité. Des ouvertures d'établissements, changements de collecte, dépendances entre événements ou saisons peuvent expliquer des différences.

## 1. Goods receipts

### Audit

610 lignes uniques après le nettoyage précédent (612 brutes). Les dates et quantités sont exploitables. Quatre inspections restent inconnues, mais `receipt_status` est renseigné pour chaque réception et définit le taux étudié.

### Fréquences

`FCL-1011` compte 133 réceptions, devant `FCL-1007` (76) et `FCL-1010` (61). Ce classement mesure l'activité enregistrée, pas un dysfonctionnement.

IQR/Z signalent **113 mois distincts**, principalement à cause des comptes très faibles et de nombreuses valeurs zéro. Ce grand nombre n'est pas traité comme une preuve de 113 problèmes.

Le contrôle conditionnel après BH retient **3 pics**, tous à `FCL-1011` :

| Mois | Réceptions | q ajusté BH |
|---|---:|---:|
| Mai 2018 | 6 | ≈ 0,000353 |
| Juin 2021 | 9 | ≈ 0,000353 |
| Mars 2022 | 9 | ≈ 0,021735 |

Les identifiants des réceptions concernées sont consultables dans les tables affichées par le notebook. Vérifier si ces concentrations correspondent à une activité réelle, des saisies groupées ou une couverture particulière.

### Attentes qualité

- **34/610 = 5,57 %** des réceptions sont en `quality_hold`.
- Taux brut le plus élevé : `FCL-1023`, **2/8 = 25 %**. Ce faible dénominateur implique une forte incertitude.
- **Aucun établissement** ne présente un taux significativement supérieur aux autres au seuil q<0,05 après BH.
- Les taux trimestriels sont affichés avec effectifs et intervalles. Un trimestre sans réception a un taux manquant, pas zéro.
- Les taux par établissement **et** trimestre sont conservés dans la variable `quality_table` (aperçu affiché dans le notebook) ; une carte thermique représente les six établissements les plus documentés sur les douze derniers trimestres, avec chaque dénominateur.

**Conclusion :** trois pics de fréquence méritent une vérification documentaire. Aucun signal statistique suffisamment étayé ne permet ici de qualifier un établissement de particulièrement défaillant sur la qualité.

## 2. Inventory adjustments

### Audit et fréquences

140 lignes uniques, dont **89 diminutions** et **51 augmentations**. Un rôle d'approbation manque ; 117 références de réception et 103 références d'expédition sont vides et restent conservées.

`FCL-1010` est le plus représenté avec 20 ajustements. Cela ne fournit aucun taux rapporté au stock ou aux réceptions : le dénominateur d'activité n'est pas dans ce fichier.

31 mois sont signalés par le Z-score descriptif, mais **aucun pic de fréquence ne subsiste au seuil q<0,05** après le test conditionnel et BH. Les signaux de Z sur des séries très creuses doivent être interprétés avec prudence.

### Quantités : trois lectures à distinguer

1. Quantité signée par catégorie/unité : les deux queues peuvent inclure des augmentations rares dans un groupe dominé par les diminutions.
2. Amplitude absolue par catégorie/unité : recherche d'ajustements particulièrement importants ou petits.
3. Vérification secondaire par catégorie/unité/signe, uniquement pour n≥8 : vérifie la sensibilité au mélange des directions.

L'union de ces lectures signale **23 ajustements**, ce qui ne veut pas dire 23 mouvements excessifs ou erreurs.

**7 ajustements ont une amplitude particulièrement élevée au sein de leur catégorie/unité**, tous en `medical_consumables / case` :

| Ajustement | Quantité signée (case) |
|---|---:|
| ADJ-2019-00032 | −5,86 |
| ADJ-2019-00049 | −5,45 |
| ADJ-2021-00068 | −5,44 |
| ADJ-2022-00093 | +7,41 |
| ADJ-2022-00097 | −7,88 |
| ADJ-2024-00116 | −6,68 |
| ADJ-2025-00132 | +5,55 |

**Conclusion :** la structure dominante des consommables médicaux (nombreuses petites diminutions) explique une partie des signaux. Relire les sept amplitudes, leurs motifs et les conventions de stock ; ne pas inverser un signe parce qu'un motif paraît surprenant.

## Lecture des tableaux affichés dans les notebooks

Pour chaque dataset, le registre `register` affiché dans le notebook contient les détections, scores, bornes, groupe de référence, méthode et interprétation possible. Les règles et seuils sont affichés à la suite du registre (aperçu des premières lignes). Aucun fichier CSV d’anomalies ou de seuils n’est créé.

- `echelle_seuil=valeur` : les bornes concernent la mesure brute (IQR, Z et Z modifié).
- `echelle_seuil=score` : lire la règle et le score (q<0,05 pour les tests exacts).
- Les identifiants `FCL-…|AAAA-MM` sont des mois-établissement ; `ADJ-…` désigne un ajustement individuel. Ne pas les additionner comme des unités équivalentes.
- Une observation peut être signalée plusieurs fois par des méthodes différentes. Les 100 lignes du registre d'ajustements correspondent à 23 ajustements et 31 mois distincts, pas 100 événements anormaux.

## Limites et points à vérifier

- Confirmer la couverture continue des fichiers et l'exposition réelle de chaque établissement ; la première/dernière observation n'est pas une date d'ouverture/fermeture.
- Les tests supposent une stabilité locale et des événements indépendants. Les fenêtres se chevauchent ; les résultats restent exploratoires même après BH.
- Les seuils IQR/Z ne sont pas ajustés pour tests multiples et sont sensibles aux séries creuses, aux mixtures de signes et aux petits effectifs.
- La qualité peut dépendre des marchandises, fournisseurs et périodes ; Fisher ne corrige pas ces facteurs.
- IF/LOF ne sont pas imposés : pour les ajustements, quantité et amplitude sont redondantes ; pour les taux, les dénominateurs doivent être explicitement traités. DBSCAN ou un encodage numérique des identifiants ne résoudraient pas ces problèmes.
- Aucune incohérence entre fichiers n'est ajoutée à ces résultats autonomes. Le notebook local de jointures est laissé intact pour une étape ultérieure.

## Vérification technique et transparence

Exécution intégrale des deux notebooks avec des noyaux Jupyter neufs ; graphiques et résultats enregistrés. Assertions sur les sommes des comptes mensuels, les dénominateurs, les clés et l'éligibilité des mois détectés. Versions : Python 3.9, pandas 2.3.3, numpy 2.0.2, scipy 1.13.1, scikit-learn 1.6.1, matplotlib 3.9.4.



## Complément du 10 novembre 2026 — contrôles documentaires ciblés

**Réceptions et commandes.** Jointure sur `(purchase_order_number, purchase_order_line)` avec le fichier de commandes nettoyé : **0 réception sans commande correspondante** et **0 désaccord d'unité**. Après cumul des réceptions par ligne de commande, **65 lignes** présentent une quantité reçue inférieure à la quantité commandée, **aucune** une quantité supérieure. Il s'agit des quantités *enregistrées dans l'extraction*, pas de la preuve d'un défaut de livraison ; tenir compte de l'état de commande et des éventuelles livraisons ultérieures. La table `inspection_status × receipt_status` est cohérente dans l'extraction : 34 `quality_hold` correspondent à 34 inspections `held` ; les 4 inspections absentes sont associées à `posted` et restent inconnues.

**Ajustements et événements associés.** Sur 140 ajustements, **23 références** de réception et **37 références** d'expédition sont renseignées et toutes retrouvées. Parmi elles, **11 réceptions** et **13 expéditions** sont datées après l'ajustement qui les cite. Les deux familles se chevauchent : **21 ajustements distincts** possèdent au moins une de ces références futures. Ce constat est potentiellement important pour la traçabilité et la chronologie, mais le sens métier exact de `related_receipt_number` / `related_shipment_number` doit être clarifié avant de parler de lien impossible ou de correction frauduleuse. Les comparaisons sont établies sur la date (pas l'heure) de l'ajustement et le premier départ enregistré d'une expédition.

**Étape suivante pour le groupe.** Relire les 21 ajustements avec leurs motifs et statuts, vérifier les références atypiques avec les documents, et discuter de `FCL-1029` (présent dans les ajustements et le référentiel brut, absent du référentiel nettoyé). Le notebook local `analyse_logistics.ipynb` centralise les contrôles de catégorie, unité et établissement, mais ses conclusions restent **des indices documentaires**, et non des accusations.


## Complément méthodologique — cours 2 et 3 (10 novembre 2026)

**Clustering descriptif des 30 établissements de réception (cours 2).** Quatre variables à l'échelle de l'établissement sont utilisées : nombre de réceptions, mois d'activité, réceptions moyennes pendant les mois actifs et proportion de `quality_hold`. Après transformation logarithmique des volumes et standardisation, les silhouettes K-means pour k=2, 3, 4 sont **0,456 ; 0,488 ; 0,478**. Le choix exploratoire **k=3** distingue 16 établissements peu actifs (médiane 5,5 réceptions), 9 plus actifs (médiane 37) et 5 de moindre volume mais à taux médian d'attente qualité plus élevé (14,3 %). Ce dernier groupe comprend de petits dénominateurs : il **ne prouve pas** une mauvaise qualité et ne remplace pas les tests exacts de Fisher.

**Règles d'association sur les 140 ajustements (cours 3).** Un panier comporte les modalités de motif, catégorie, signe et statut. Recherche de couples fréquents (support ≥ 8 %, confiance ≥ 60 %, lift ≥ 1,2) : **six règles orientées** passent les seuils exploratoires. L'association `statut=approved → signe=augmentation` concerne 17 ajustements, support 12,1 %, confiance 68 %, lift ≈ 1,87. L'association `categorie=monitoring_equipment → signe=augmentation` concerne 12 ajustements, support 8,6 %, confiance 60 %, lift ≈ 1,65. Ces régularités décrivent des modalités corrélées ; elles ne démontrent pas d'irrégularité, d'intention ou de causalité et nécessitent une lecture métier des statuts.
