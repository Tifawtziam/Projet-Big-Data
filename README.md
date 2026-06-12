# 📊 Projet Big Data — Optimisation de la Consommation Énergétique

## 🗂️ Structure du projet

```
├── notebooks/          ← Notebooks Azure Synapse / Apache Spark
├── powerbi/
│   ├── dashboard.pbix  ← Fichier Power BI Desktop
│   └── screenshots/    ← Captures PNG des dashboards
└── README.md
```

## ⚠️ Note sur les données

Les données sont stockées sur Azure Data Lake Storage Gen2.
Les crédits Azure étudiants ont expiré — le Data Lake n'est plus accessible.

Les fichiers de données sont disponibles ici :
👉 [Kaggle - Smart Meters in London](https://www.kaggle.com/datasets/jeanmidev/smart-meters-in-london)


## 🛠️ Stack technique

- **Azure Synapse Analytics** — Orchestration et traitement des données
- **Apache Spark** — Traitement distribué
- **Delta Lake** — Stockage des données structurées
- **Azure Machine Learning** — Entraînement des modèles Prophet et LSTM
- **Power BI** — Visualisation et dashboards

---

## 📈 Analyse des Dashboards Power BI

### Graphique 1 — Évolution de la consommation dans le temps

> **Type :** Courbe (Line Chart) | **Axes :** `tstp` (X) / `energy_kwh` (Y)

Le graphique en barres groupées superposées montre l'évolution de `energy_kwh` sur la période du **24 au 27 décembre 2012**, découpée en intervalles fins (niveaux de consommation de 0.0 à 0.014 kWh).

**Observations :**
- La consommation présente une **variabilité journalière marquée** : les 25 et 26 décembre montrent des pics bien supérieurs aux 24 et 27.
- On observe un **pic notable le 25 décembre** (environ 50 mesures groupées), correspondant à une forte activité domestique liée aux fêtes.
- Les périodes de **faible consommation** se concentrent sur les valeurs proches de 0.0–0.003 kWh, indiquant des moments d'inactivité ou de nuit.
- La courbe multi-lignes confirme ces **comportements saisonniers** : les profils de consommation par tranche évoluent différemment selon les jours, certaines tranches (0.012 kWh) montant fortement le 25 avant de redescendre.

---

### Graphique 2 — Consommation réelle vs prédite ⭐

> **Type :** Courbe | **Axes :** `tstp` (X) / `energy_kwh` et `prediction` (Y)

C'est le **graphique le plus stratégique du projet**, car il permet de juger directement la qualité du modèle Prophet.

**Observations :**
- Sur la plupart des jours, les valeurs réelles et prédites sont **relativement proches**, témoignant d'une bonne capacité prédictive générale de Prophet.
- Cependant, le **25 décembre** montre un écart visible : les prédictions sous-estiment légèrement la consommation réelle sur certaines tranches, indiquant que Prophet a du mal à capter les **hausses brusques liées aux événements atypiques** (jours fériés, comportements exceptionnels).
- Le modèle **suit correctement les tendances générales** mais peut montrer des décalages sur les microcomportements intra-journaliers.

---

### Graphique 3 — Consommation par heure

> **Type :** Histogramme / Colonne | **Axes :** `hour` (X) / Moyenne de `energy_kwh` (Y)

**Observations :**
- La courbe présente un profil en **deux paliers très nets** :
  - **Heure 0 à ~1h** : consommation élevée (~8 kWh) → **pic nocturne** (équipement à démarrage automatique ou recharge de véhicule électrique)
  - **Heures 1h à 23h** : chute brutale et maintien autour de **6 kWh** → consommation de base en journée
- **Heures de pointe** : autour de minuit / 1h du matin.
- **Heures creuses** : la journée entière présente une consommation relativement stable et basse.
- Ce profil inhabituel mérite attention : il peut indiquer un équipement spécifique ou une **anomalie de mesure** à investiguer.

---

### Graphique 4 — is_peak (périodes de pointe tarifaire)

> **Type :** Histogramme | **Axes :** `is_peak` (X) / Moyenne de `energy_kwh` (Y)

**Observations :**
- Le graphique montre une **relation inverse claire** :
  - `is_peak = 0` (hors pointe) → consommation moyenne très élevée (~95 kWh)
  - `is_peak = 1` (en pointe tarifaire) → consommation nettement plus basse (~50 kWh)
- Ce résultat est **contre-intuitif au premier abord**, mais s'explique par le fait que les utilisateurs **adaptent leur comportement** en période de pointe (effacement, report de charges) — ce qui est exactement l'objectif d'un projet d'optimisation énergétique.
- Cela valide l'utilité de la variable `is_peak` comme **feature discriminante** dans le modèle de prédiction.

---

### Graphique 5 — Erreur du modèle (MAE et RMSE par modèle)

> **Type :** Histogramme groupé | **Modèles comparés :** Prophet vs LSTM

| Métrique | Prophet | LSTM |
|----------|---------|------|
| MAE      | ≈ 0.021 | ≈ 0.013 |
| RMSE     | ≈ 0.025 | ≈ 0.017 |

**Observations :**
- Le modèle **LSTM surpasse Prophet** sur les deux métriques, avec des erreurs environ **35–40% inférieures**.
- Prophet prédit bien les **tendances globales** (comportements saisonniers, cycles hebdomadaires), mais LSTM capture mieux les **dépendances temporelles fines** et les patterns à court terme.
- Prophet prédit **moins bien** lors des événements atypiques (jours fériés, changements brusques), là où LSTM, grâce à sa mémoire séquentielle, reste plus précis.
- **Recommandation** : LSTM pour l'optimisation en temps réel ; Prophet comme baseline interprétable pour les prévisions à long terme.

---

## 🔍 Synthèse

| Graphique | Insight clé |
|-----------|-------------|
| Évolution temporelle | Forte variabilité jour/nuit, pics liés aux événements (fêtes) |
| Réel vs Prédit | Prophet suit les tendances mais sous-estime les pics atypiques |
| Consommation horaire | Pic nocturne à 0–1h, journée stable → optimisation possible la nuit |
| is_peak | Les utilisateurs réduisent la consommation en heures de pointe (comportement adaptatif) |
| MAE / RMSE | LSTM > Prophet en précision ; Prophet utile pour l'interprétabilité |

---

## 👥 Auteurs

> Projet réalisé dans le cadre du cours Big Data & Cloud Computing.
