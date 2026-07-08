# Analyse et Modélisation de la Demande d'Électricité en Suisse

Ce dépôt rassemble les travaux de modélisation statistique et de prévision de la demande électrique mensuelle en Suisse (période 2017-2025). Le projet est structuré en deux phases clés : une analyse rigoureuse de la stationnarité et des propriétés structurelles de la série, suivie d'une phase de modélisation stochastique par approche SARIMA pour générer des prévisions robustes.

## Rapports Interactifs (GitHub Pages)

Les résultats complets, incluant les graphiques dynamiques, les diagnostics de résidus et les visualisations des prévisions, sont consultables directement :

* **[Rapport Partie 1 : Étude de Stationnarité et Tests de Racine Unitaire](https://pierrickvernet.github.io/time-series-electricity-demand-forecasting-Switzerland/part%201/time_series_switzerland_part_1.html)**
* **[Rapport Partie 2 : Identification, Estimation et Prévisions SARIMA](https://pierrickvernet.github.io/time-series-electricity-demand-forecasting-Switzerland/part%202/time_series_switzerland_part_2.html)**

---

## Synthèse de la Démarche Statistique

### Partie 1 : Caractérisation de la Série Temporelle
Avant toute modélisation prédictive, les propriétés stochastiques de la demande brute ont été disséquées :
* **Analyse Descriptive :** Mise en évidence d'un profil de consommation mature sous forme de plateau (moyenne mensuelle de 5,68 TWh), caractérisé par une forte saisonnalité hivernale (pics à 7 TWh) liée aux contraintes thermiques (chauffage).
* **Stratégie de Différenciation :** Application d'un filtre de différence saisonnière d'ordre 12 ($d=0, D=0$ avec différenciation à l'ordre 12, notée $\Delta_{12} X_t$) pour stabiliser la série et éliminer la composante périodique.
* **Tests de Racine Unitaire :** Déploiement d'une batterie de tests pour valider la stationnarité de la série désaisonnalisée :
    * *Dickey-Fuller Augmenté (ADF)* : Rejet de l'hypothèse nulle de racine unitaire en spécification `none` (statistique de -3,70 vs valeur critique de -1,95).
    * *Zivot-Andrews & Lee-Strazicich* : Modélisation des chocs potentiels (confinements COVID-19, crise énergétique de 2022).
    * Les résultats confirment l'absence de rupture structurelle statistiquement significative sur le niveau ou la tendance.
  
**La série désaisonnalisée suit un processus purement stationnaire.**

### Partie 2 : Modélisation SARIMA et Prévisions
La série $\Delta_{12} X_t$ étant stationnaire, l'analyse des fonctions d'autocorrélation (ACF/PACF) et de la matrice EACF a guidé la sélection des architectures stochastiques.
* **Modèles Estimés :** Évaluation compétitive de 5 spécifications SARIMA (avec élimination progressive des coefficients non significatifs par sélection descendante/*backward*).
* **Diagnostics des Résidus :** Validation rigoureuse des hypothèses fondamentales sur les erreurs du modèle optimal :
    * *Normalité* : Validée par le test de Jarque-Bera ($p$-value = 0,9949).
    * *Absence d'autocorrélation* : Validée par le test de Ljung-Box sur 24 retards ($p$-value = 0,2469).
    * *Espérance nulle* : Validée par Student $t$-test ($p$-value = 0,6175).
    * *Homoscédasticité* : Validée par le test d'Engle-ARCH ($p$-value = 0,6662).

---

## Performances des Modèles Évalués

Les critères d'information d'Akaike (AIC) et bayésien (BIC) ont été utilisés pour arbitrer le compromis biais-variance. Le tableau suivant synthétise les performances des spécifications testées sur la série brute :

| Modèle Évalué | Spécification Finale | AIC | BIC | Statut des Résidus |
| :--- | :--- | :---: | :---: | :--- |
| **Modèle 1** | $SARIMA(0,0,3)(0,0,1)_{12}$ | 9,06 | 24,11 | Non Bruit Blanc (Autocorrélé) |
| **Modèle 2** | $SARIMA(1,0,3)(0,0,1)_{12}$ (contraint) | 9,06 | 24,11 | Hétéroscédastique (Effet ARCH) |
| **Modèle 3** | $SARIMA(2,0,2)(0,0,1)_{12}$ | 11,04 | 28,58 | Non Bruit Blanc (Autocorrélé) |
| **Modèle 4** | **$SARIMA(12,0,0)(0,0,1)_{12}$ (contraint)** | **-27,81** | **-3,84** | **Validé (Bruit Blanc Gaussien)** |
| **Modèle 5** | $SARIMA(0,0,9)(0,0,1)_{12}$ (contraint) | 26,10 | 46,16 | Non Bruit Blanc (Autocorrélé) |

Le **Modèle 4** s'impose comme l'unique spécification valide. Ses critères d'information négatifs traduisent une nette supériorité en termes de parcimonie et de qualité d'ajustement.

---

## Prévisions à 12 mois

Le modèle retenu a permis de générer les trajectoires de consommation sur un horizon de un an (12 mois). Grâce à la validation des hypothèses de normalité et d'homoscédasticité, les intervalles de confiance à 95% sont mathématiquement fiables et permettent de borner précisément le risque d'approvisionnement. Les projections reproduisent fidèlement les dynamiques de transition saisonnière (pics hivernaux et creux estivaux) propres au réseau électrique suisse.

---

## Structure du Répertoire

```text
├── part 1/
│   └── time_series_switzerland_part_1.html   # Rapport HTML - Analyse et Tests de Stationnarité
├── part 2/
│   └── time_series_switzerland_part_2.html   # Rapport HTML - Modélisation et Prévisions
├── monthly_full_release_long_format(1).csv   # Données historiques de la demande énergétique
└── README.md                                 # Synthèse du projet
```

🛠️ Stack Technique R

Le code source s'appuie sur les bibliothèques standards de l'analyse des séries temporelles et de l'économétrie :
forecast & stats : Estimation des processus SARIMA et génération des prévisions.
urca, tseries & CADFtest : Implémentation des tests ADF, Zivot-Andrews et calcul des statistiques de racine unitaire.
lmtest & FinTS : Tests de significativité des coefficients (coeftest) et détection des effets ARCH (ArchTest).
TSA & knitr : Matrice EACF pour l'identification et mise en forme des tableaux statistiques.

