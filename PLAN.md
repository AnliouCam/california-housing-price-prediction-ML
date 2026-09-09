# Plan — Projet Examen Machine Learning

Sujet retenu : **C — Estimation de la valeur d'un bien immobilier** (California Housing, `sklearn.datasets.fetch_california_housing`)

Choisi pour la rapidité : dataset intégré à scikit-learn, aucune variable catégorielle, aucune valeur manquante, aucun déséquilibre de classes à gérer.

Deadline projet : fin de semaine 2 (à confirmer avec l'enseignant). Soutenance début semaine 3.

## Suivi d'avancement

- [x] 1. Chargement + EDA (statistiques, distributions, corrélations)
- [x] 2. Split train/test + prétraitement (outliers, standardisation)
- [x] 3. Modélisation (Ridge/Lasso, RandomForest, KNN)
- [x] 4. Recherche d'hyperparamètres (GridSearchCV / RandomizedSearchCV)
- [x] 5. Évaluation finale (RMSE, MAE, R², analyse des résidus)
- [x] 6. Interprétabilité (feature importance / SHAP)
- [x] 7. Analyse critique (limites, biais, pistes d'amélioration) — brouillon rédigé, à relire/personnaliser
- [ ] 8. Rapport écrit (PDF, 8-12 pages)
- [~] 9. README + requirements.txt (fait) + dépôt Git (reste à faire)
- [ ] 10. Slides soutenance (8-10 diapos)

## Correspondance avec la grille de notation (/20)

| Critère | Points | Étape du plan |
|---|---|---|
| EDA + prétraitement | 3 | 1, 2 |
| Modélisation | 4 | 3 |
| Rigueur évaluation | 4 | 4, 5 |
| Interprétabilité + analyse critique | 3 | 6, 7 |
| Qualité code / Git | 2 | 9 |
| Rapport écrit | 2 | 8 |
| Soutenance orale | 2 | 10 |

## Décisions prises (à pouvoir justifier à l'oral)

- Dataset : California Housing plutôt que Ames Housing → moins de prétraitement, gain de temps, toujours conforme au sujet C.
- 3 modèles : Ridge (linéaire régularisé), RandomForest (ensemble d'arbres), KNN (3e modèle de comparaison — attendu qu'il soit moins performant, c'est normal et à expliquer).
- Outil IA utilisé : Claude Code, pour la génération du squelette de notebook et l'aide à la rédaction. À mentionner dans la section "Utilisation d'outils d'IA" du rapport (obligatoire, section 6 du sujet).

## Observations pour le rapport (à réutiliser en section EDA / analyse critique / interprétabilité)

- Target `MedHouseVal` exprimée en centaines de milliers de $ (confirmé dans `DESCR` officiel de sklearn). `5.0` = 500 000 $.
- Pic anormal de `MedHouseVal` à la valeur max (5.0) sur l'histogramme → observation empirique suggérant un plafonnement des prix élevés lors du recensement de 1990. À formuler comme hypothèse/observation, pas comme fait officiellement documenté par sklearn.
- `HouseAge` : pic à 52 → probable plafonnement de l'âge max dans les données source.
- `AveRooms`, `AveBedrms`, `Population`, `AveOccup` : distributions très asymétriques, valeurs aberrantes extrêmes visibles sur les histogrammes → à traiter en prétraitement (section 3.2 du sujet), à mentionner en limites/biais.
- Corrélations avec `MedHouseVal` : `MedInc` = 0.69 (variable la plus prédictive) ; `AveRooms` = 0.15 ; `Latitude` = -0.14 ; le reste proche de 0.
- Colinéarité forte : `AveRooms` / `AveBedrms` = 0.85 ; `Latitude` / `Longitude` = -0.92 (géographiquement logique). À mentionner pour justifier pourquoi Ridge (régularisation) est pertinent face à cette colinéarité, et pourquoi RandomForest y est peu sensible.
- `Latitude`/`Longitude` : deux pics de densité (~34 et ~38) correspondant aux zones de Los Angeles et San Francisco/Bay Area — cohérent avec la géographie réelle de la Californie.
- Dataset basé sur le recensement 1990 → biais temporel à mentionner en section 7 (analyse critique) : prix et structure du marché immobilier californien ont beaucoup changé depuis, le modèle ne serait pas directement utilisable pour une estimation actuelle sans réentraînement sur données récentes.

- Outliers `AveRooms`/`AveBedrms`/`Population`/`AveOccup` : traités par **capping IQR (k=1.5)** plutôt que suppression, car la doc officielle sklearn confirme que ce sont de vraies observations (quartiers avec peu de foyers/beaucoup de logements vides, ex. zones de vacances) et non des erreurs. Bornes calculées sur train uniquement, appliquées à train+test (pas de fuite de données). Ex: `AveOccup` capé à [1.15, 4.56] (max avant capping : 1243).

- Résultats baseline (CV 5-fold sur train, avant tuning) : Ridge RMSE=0.665/R²=0.669 ; **RandomForest RMSE=0.512/R²=0.804 (meilleur)** ; KNN RMSE=0.624/R²=0.709. RandomForest nettement meilleur. KNN devance Ridge — à expliquer en analyse critique (relations géographiques probablement mieux captées par similarité locale que par un modèle linéaire).

- Tuning : Ridge (`alpha=0.1`, GridSearchCV, gain quasi nul) ; RandomForest (`n_estimators=200, max_depth=None, min_samples_leaf=2, max_features='sqrt'`, RandomizedSearchCV 10 tirages car espace trop grand pour grille exhaustive, RMSE 0.512→**0.5005**) ; KNN (`n_neighbors=15, weights='distance'`, GridSearchCV, RMSE 0.624→0.602). **RandomForest tuné retenu comme modèle final.**

- Évaluation finale (test, RandomForest tuné) : RMSE=0.497, MAE=0.330, R²=0.811. Résidus centrés (~0), pas de surapprentissage visible.
- **Découverte clé (à réutiliser en analyse critique 3.7/section 7)** : ligne verticale visible sur "prédictions vs réel" à y=5.0 → confirme empiriquement le plafonnement des prix élevés. RMSE local sur observations plafonnées (y_test≥4.9, n=190) = 1.14, soit 2.3x pire que le RMSE global. C'est LA limite concrète et chiffrée du modèle à mettre en avant à l'oral.

- Interprétabilité : feature importance native + permutation importance (pas de SHAP, pour rester rapide — les deux méthodes sklearn suffisent et sont cohérentes entre elles). Classement : `MedInc` (0.37/0.51) >> `Latitude`/`Longitude` (0.13-0.14/0.27-0.32) > `AveOccup`/`AveRooms` > `HouseAge` > `AveBedrms`/`Population` (quasi nul). Lat/Long importantes en RF malgré faible corrélation linéaire → confirme la non-linéarité géographique, explique pourquoi RF/KNN battent Ridge.

## Notes / blocages

_(à compléter au fil de l'avancement)_
