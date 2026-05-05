# Description du jeu de données — Pharma Sales Data

## Source et licence

- **Auteur :** Milan Zdravković
- **Source originale :** [Kaggle — Pharma Sales Data](https://www.kaggle.com/datasets/milanzdravkovic/pharma-sales-data)
- **Notebook de référence :** [Pharma sales data analysis and forecasting](https://www.kaggle.com/code/milanzdravkovic/pharma-sales-data-analysis-and-forecasting)
- **Licence :** Creative Commons Attribution-NonCommercial 4.0 International (CC BY-NC 4.0)

## Contexte

Données de ventes issues du **système de point-of-sale d'une seule pharmacie**. Les ventes individuelles (57 médicaments distincts) ont été agrégées selon la classification **ATC (Anatomical Therapeutic Chemical)** en **8 catégories thérapeutiques**, puis **rééchantillonnées** à différentes granularités temporelles. Les valeurs nulles aberrantes ont été imputées en consultation avec le personnel de la pharmacie.

## Période couverte

- **Début :** janvier 2014
- **Fin :** 8 octobre 2019
- **Couverture :** 6 années dont la dernière (2019) est **incomplète** — point d'attention pour les modèles de prévision (la dernière année peut introduire un biais si on ne la tronque pas).

## Les 8 catégories ATC (variables cibles communes aux 4 fichiers)

| Code ATC | Famille thérapeutique |
|---|---|
| **M01AB** | Anti-inflammatoires non stéroïdiens (AINS) — dérivés de l'acide acétique (ex : diclofénac, indométacine) |
| **M01AE** | Anti-inflammatoires non stéroïdiens (AINS) — dérivés de l'acide propionique (ex : ibuprofène, naproxène) |
| **N02BA** | Antalgiques / antipyrétiques — dérivés de l'acide salicylique (ex : aspirine) |
| **N02BE** | Antalgiques / antipyrétiques — pyrazolones et anilides (ex : paracétamol) |
| **N05B** | Psycholeptiques — anxiolytiques |
| **N05C** | Psycholeptiques — hypnotiques et sédatifs |
| **R03** | Médicaments des syndromes obstructifs des voies aériennes (asthme, BPCO) |
| **R06** | Antihistaminiques à usage systémique (allergies) |

> **Note importante sur les valeurs :** ce ne sont **pas** des nombres entiers d'unités vendues. Les ventes individuelles ont été agrégées avec une pondération (probablement la dose ou le conditionnement), ce qui produit des **valeurs flottantes** (ex : 1384.94, 1044.24). Pour un usage pédagogique, on peut les considérer comme un **volume de ventes agrégé** par catégorie sur la période.

## Les 4 fichiers CSV

Tous les fichiers partagent la même colonne de date `datum` et les mêmes 8 colonnes de ventes (M01AB, M01AE, N02BA, N02BE, N05B, N05C, R03, R06). Ils ne diffèrent que par la **granularité temporelle** et les **colonnes de décomposition de la date** ajoutées.

---

### 1. `saleshourly.csv` — Granularité **horaire**

- **Volume :** ~50 000 lignes (~2,5 Mo) — une ligne par heure de la période d'ouverture de la pharmacie.
- **Granularité :** la plus fine du jeu de données.
- **Colonnes :**
  - `datum` — horodatage complet (date + heure)
  - `Year`, `Month`, `Hour`, `Weekday Name` — décomposition de la date
  - `M01AB`, `M01AE`, `N02BA`, `N02BE`, `N05B`, `N05C`, `R03`, `R06` — ventes agrégées sur l'heure
- **Cas d'usage pédagogiques :**
  - Étude des **patterns intra-journaliers** (heures de pointe, ouverture/fermeture).
  - Comparaison **matin vs après-midi**, **week-end vs semaine**.
  - Modélisation de séries à **double saisonnalité** (heure du jour × jour de la semaine) — particulièrement adaptée à Prophet.
- **Limites :** beaucoup de zéros (heures de fermeture), ce qui complique certains tests paramétriques et certains modèles ARIMA.

---

### 2. `salesdaily.csv` — Granularité **journalière**

- **Volume :** **2 106 lignes** (1 ligne par jour, du 02/01/2014 au 08/10/2019).
- **Granularité :** la plus utilisée pour les analyses descriptives et inférentielles.
- **Colonnes :**
  - `datum` — date au format `YYYY-MM-DD`
  - `Year`, `Month`, `Hour` (toujours 0 dans ce fichier), `Weekday Name` (lundi → dimanche en anglais)
  - `M01AB`, `M01AE`, `N02BA`, `N02BE`, `N05B`, `N05C`, `R03`, `R06` — ventes agrégées sur la journée
- **Cas d'usage pédagogiques :**
  - **Stats descriptives :** distribution des ventes journalières par catégorie, comparaison de dispersion.
  - **Inférence :** tests "week-end vs semaine", "hiver vs été", ANOVA entre catégories.
  - **Probabilités :** ajustement empirique des ventes journalières d'une catégorie à des lois usuelles (Poisson pour les catégories rares, log-normale pour les autres).
  - **Régression :** `ventes ~ jour_semaine + mois + catégorie`.
- **Point d'attention :** l'article original signale la présence de **lignes avec des zéros** sur certaines catégories. Pour la modélisation, deux options classiques : exclure ces lignes ou imputer par la moyenne/médiane.

---

### 3. `salesweekly.csv` — Granularité **hebdomadaire**

- **Volume :** ~300 lignes (1 ligne par semaine sur ~6 ans).
- **Granularité :** intermédiaire — lisse le bruit journalier tout en conservant la saisonnalité annuelle.
- **Colonnes :**
  - `datum` — date du début de semaine
  - `M01AB`, `M01AE`, `N02BA`, `N02BE`, `N05B`, `N05C`, `R03`, `R06` — ventes agrégées sur la semaine
- **Cas d'usage pédagogiques :**
  - **Premier contact avec les séries temporelles** : assez de points pour les modèles, peu pour avoir une vue d'ensemble.
  - **Décomposition STL** lisible (saisonnalité annuelle de période 52).
  - **Forecasting :** comparaison Holt-Winters / SARIMA / Prophet sur une cible plus stable que le journalier.

---

### 4. `salesmonthly.csv` — Granularité **mensuelle**

- **Volume :** ~70 lignes (1 ligne par mois sur ~6 ans, soit ~70 mois).
- **Granularité :** la plus agrégée — adaptée au reporting business et aux prévisions à moyen terme.
- **Colonnes :**
  - `datum` — premier jour du mois
  - `M01AB`, `M01AE`, `N02BA`, `N02BE`, `N05B`, `N05C`, `R03`, `R06` — ventes agrégées sur le mois
- **Cas d'usage pédagogiques :**
  - **Saisonnalité annuelle très visible** : R06 (antihistaminiques) explose au printemps, R03 (asthme) à l'automne, N02BE (paracétamol) en hiver.
  - **SARIMA** avec saisonnalité (P, D, Q, **s=12**) — exemple canonique en cours.
  - **Prophet** avec saisonnalité annuelle et jours fériés.
  - **Régression linéaire** des ventes mensuelles vs mois (effet saisonnier) et vs année (tendance).
- **Limite :** **70 points seulement** — peu de marge pour des modèles complexes, mais c'est précisément ce qui en fait un bon support pédagogique (on voit tout d'un coup d'œil).

---

## Tableau récapitulatif

| Fichier | Lignes (≈) | Granularité | Colonnes date supplémentaires | Usage privilégié |
|---|---|---|---|---|
| `saleshourly.csv` | 50 000 | Heure | `Year`, `Month`, `Hour`, `Weekday Name` | Patterns intra-jour, double saisonnalité |
| `salesdaily.csv` | 2 106 | Jour | `Year`, `Month`, `Hour`, `Weekday Name` | Descriptif, inférentiel, régression |
| `salesweekly.csv` | 300 | Semaine | (aucune) | Décomposition STL, forecasting lisse |
| `salesmonthly.csv` | 70 | Mois | (aucune) | SARIMA s=12, Prophet, reporting |

## Choix du fichier selon le module de formation

| Partie du cours | Fichier recommandé | Pourquoi |
|---|---|---|
| Statistiques descriptives | `salesdaily.csv` | Suffisamment de points pour des distributions et boxplots informatifs ; permet d'utiliser `Weekday Name`. |
| Probabilités (ajustement de lois) | `salesdaily.csv` | Volume suffisant pour comparer empirique vs théorique (Poisson, log-normale). |
| Statistiques inférentielles | `salesdaily.csv` | Permet des tests "week-end vs semaine", ANOVA mensuelle, régressions multi-variables. |
| Séries temporelles — initiation | `salesmonthly.csv` | Saisonnalité annuelle évidente, peu de bruit, idéal pour SARIMA(p,d,q)(P,D,Q,12). |
| Séries temporelles — approfondi | `salesweekly.csv` ou `saleshourly.csv` | Plus de complexité (saisonnalités multiples, plus de points pour le backtesting). |
| Détection d'anomalies | `salesdaily.csv` ou `saleshourly.csv` | Granularité fine nécessaire pour repérer des pics ponctuels. |

## Points d'attention pour la préparation des données

1. **Convertir `datum` en datetime** systématiquement (`pd.to_datetime`) et le passer en index avant toute analyse temporelle.
2. **Tronquer 2019 au 30 septembre** si on veut éviter les biais d'année incomplète sur les modèles mensuels.
3. **Gérer les zéros** : comprendre s'ils représentent une vraie absence de vente ou une donnée manquante (l'article original confirme une **vraie absence** dans la plupart des cas — pharmacie fermée, rupture, etc.).
4. **Vérifier la fréquence** après chargement avec `df.asfreq('D')` (ou 'W', 'MS', 'H') — utile pour révéler des trous éventuels.
5. **Pour SARIMA :** travailler en mensuel avec `s=12`, en hebdomadaire avec `s=52` (attention : SARIMA gère mal les périodes longues, Prophet est souvent préférable au-delà de s=24).
