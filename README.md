# California Housing Price Prediction

Sujet C : estimation de la valeur d'un bien immobilier (dataset California Housing).

## Prérequis

- Python **3.14** (version utilisée pour développer et tester ce projet). Une version 3.11+ devrait fonctionner mais n'a pas été testée.
- Git (pour cloner le dépôt).

## Installation

```bash
git clone <url-du-depot>
cd Projet_Examen_ML

# Optionnel mais recommandé : environnement virtuel isolé
python -m venv .venv
.venv\Scripts\activate      # Windows
# source .venv/bin/activate   # macOS/Linux

pip install -r requirements.txt
```

## Reproduire les résultats

1. Ouvrir `projet_ml.ipynb` dans VS Code (extensions **Python** et **Jupyter** requises) ou dans Jupyter Notebook/JupyterLab classique.
2. Sélectionner l'interpréteur Python où `requirements.txt` a été installé.
3. Exécuter toutes les cellules dans l'ordre (`Run All`).

Le dataset (`California Housing`) est téléchargé automatiquement au premier appel de `fetch_california_housing()` (mis en cache localement ensuite, pas besoin de fichier de données séparé).

## Astuce

Au cas où une cellule mettrait du temps à répondre dans VS Code, il est aussi possible d'exécuter le notebook directement en ligne de commande :

```bash
python -m nbconvert --to notebook --execute --inplace projet_ml.ipynb
```

Le fichier `projet_ml.ipynb` est alors réécrit avec tous les résultats (tableaux, graphiques) déjà calculés — il suffit ensuite de le rouvrir pour les consulter, sans avoir besoin de relancer les cellules une par une.

## Structure du dépôt

```
Projet_Examen_ML/
├── projet_ml.ipynb     # Notebook principal (EDA, modélisation, évaluation, interprétabilité)
├── requirements.txt    # Dépendances Python (versions figées)
└── README.md           # Ce fichier
```

## Reproductibilité

`random_state=42` fixé partout où c'est pertinent (split train/test, modèles, recherche d'hyperparamètres) pour garantir des résultats identiques d'une exécution à l'autre.
