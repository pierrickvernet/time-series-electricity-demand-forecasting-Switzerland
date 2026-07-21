# Analyse et Modélisation de la Demande d'Électricité en Suisse

## 1. INTRODUCTION
- **Cadre de réalisation :** Projet de prévision de série temporelle, réalisé dans le cadre de mon M1 IREF .
- **Présentation du sujet :** Ce projet est consacré à la modélisation statistique et à la prévision de la demande électrique mensuelle globale en Suisse sur la période 2017-2025.
- **Intérêt et cas d'usage :** La demande électrique suisse présente une forte sensibilité aux variations thermiques (chauffage en période hivernale) et subit des variations macroéconomiques et structurelles. Disposer d'un modèle prédictif robuste permet d'anticiper les pics de charge, de borner précisément le risque d'approvisionnement et d'optimiser la gestion des réseaux de transport d'énergie.

---

## 2. SOURCES ET DONNÉES
- **Origine des données :** Registre historique de la demande énergétique mensuelle suisse (fichier `monthly_full_release_long_format(1).csv.zip` dans le répertoire `data/`).
- **Périmètre et caractéristiques :**
  - **Période d'étude :** Janvier 2017 à 2025 (données mensuelles).
  - **Moyenne mensuelle :** 5,68 TWh.
  - **Variable clé :** Demande brute mensuelle en TWh, caractérisée par une consommation mature sous forme de plateau et une forte saisonnalité annuelle (pics à environ 7 TWh en hiver).

---

## 3. MÉTHODOLOGIE ET DÉTAILS TECHNIQUES

### Pipeline d'analyse et de traitement
1. **Analyse exploratoire et désaisonnalisation :**
   - Identification de la structure saisonnière d'ordre 12.
   - Application d'un filtre de différence saisonnière ($\Delta_{12} X_t = X_t - X_{t-12}$) pour éliminer le schéma périodique annuel et stabiliser la série.
2. **Tests de stationnarité et de rupture structurelle :**
   - **Test ADF (Augmented Dickey-Fuller) :** Statistique de -3,70 contre une valeur critique de -1,95 (spécification `none`), permettant le rejet de l'hypothèse nulle de racine unitaire.
   - **Tests de Zivot-Andrews et Lee-Strazicich :** Analyse de la résilience de la série face aux chocs exogènes (confinements COVID-19, crise énergétique de 2022). Confirmation de l'absence de rupture structurelle statistiquement significative sur le niveau ou la tendance.
3. **Identification et estimation SARIMA :**
   - Inspection des fonctions d'autocorrélation (ACF/PACF) et de la matrice EACF sur la série stationnaire $\Delta_{12} X_t$.
   - Évaluation comparative de 5 architectures SARIMA avec procédure de sélection descendante (*backward*) pour écarter les coefficients non significatifs.
4. **Diagnostic rigoureux des résidus :**
   - **Normalité :** Test de Jarque-Bera ($p$-value = 0,9949).
   - **Absence d'autocorrélation :** Test de Ljung-Box sur 24 retards ($p$-value = 0,2469).
   - **Espérance nulle :** Test $t$ de Student ($p$-value = 0,6175).
   - **Homoscédasticité :** Test d'Engle-ARCH ($p$-value = 0,6662).

### Technologies et bibliothèques (Environnement R)
- **Modélisation et prévision :** `forecast`, `stats`
- **Tests économétriques et racine unitaire :** `urca`, `tseries`, `CADFtest`
- **Diagnostics et inférence :** `lmtest`, `FinTS`
- **Identification et reporting :** `TSA`, `knitr`

---

## 4. CONCLUSION ET RÉSULTATS CLÉS

### Évaluation des modèles candidats

| Modèle Évalué | Spécification Finale | AIC | BIC | Statut des Résidus |
| :--- | :--- | :---: | :---: | :--- |
| **Modèle 1** | $SARIMA(0,0,3)(0,0,1)_{12}$ | 9,06 | 24,11 | Non Bruit Blanc (Autocorrélé) |
| **Modèle 2** | $SARIMA(1,0,3)(0,0,1)_{12}$ (contraint) | 9,06 | 24,11 | Hétéroscédastique (Effet ARCH) |
| **Modèle 3** | $SARIMA(2,0,2)(0,0,1)_{12}$ | 11,04 | 28,58 | Non Bruit Blanc (Autocorrélé) |
| **Modèle 4** | **$SARIMA(12,0,0)(0,0,1)_{12}$ (contraint)** | **-27,81** | **-3,84** | **Validé (Bruit Blanc Gaussien)** |
| **Modèle 5** | $SARIMA(0,0,9)(0,0,1)_{12}$ (contraint) | 26,10 | 46,16 | Non Bruit Blanc (Autocorrélé) |

### Apports et enseignements
- Le **Modèle 4 ($SARIMA(12,0,0)(0,0,1)_{12}$ contraint)** est retenu comme l'unique spécification valide respectant l'ensemble des hypothèses sur les résidus, affichant les critères d'information (AIC = -27,81, BIC = -3,84) les plus performants.
- Les projections à 12 mois restituent fidèlement la dynamique saisonnière (pics hivernaux et creux estivaux) avec des intervalles de confiance à 95% scientifiquement étalonnés.

### Limites et perspectives
- **Limites :** Le modèle repose sur une approche purement stochastique univariée, sans intégrer de variables explicatives exogènes dynamiques (températures quotidiennes, indicateurs d'activité industrielle).
- **Perspectives :** Intégration d'un modèle ARIMAX ou SARIMAX incorporant des données météorologiques et d'un modèle de lissage exponentiel (TBATS/Prophet) à des fins de comparaison.

---

## 5. STRUCTURE DU DÉPÔT

```text
.
├── data/
│   └── monthly_full_release_long_format(1).csv.zip   # Fichier de données historique compressé
├── functions/
│   ├── LeeStrazicichUnitRootTest.R                   # Test de racine unitaire (Lee & Strazicich)
│   └── LeeStrazicichUnitRootTestParallelization.R    # Version parallélisée avec bootstrap
├── .gitattributes                                     # Configuration Git (LFS, fins de ligne)
├── README.md                                          # Documentation générale du projet
├── time_series_switzerland_part_1.Rmd                 # Code RMarkdown - Partie 1 (Stationnarité)
├── time_series_switzerland_part_1.html                # Rapport HTML compilé - Partie 1
├── time_series_switzerland_part_2.Rmd                 # Code RMarkdown - Partie 2 (Modélisation SARIMA)
└── time_series_switzerland_part_2.html                # Rapport HTML compilé - Partie 2                                       # Documentation du projet
```

---
