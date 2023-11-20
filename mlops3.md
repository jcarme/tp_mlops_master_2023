# Tests, scripts, automatisation

# Tests

Vous allez commencer par écrire un jeu de test fondé sur le _framework_ `pytest`, qui sera executé sur chaque modèle avant qu'il ne puisse être passé en production.

## Ecrire les tests

Créez un répertoire `tests` dans lequel vous allez stocker vos tests.

Rappel sur `pytest`
- Tous les fichiers de test doivent être nommés `test_*` et les fonctions de tests `test_*`
- Les tests échouent quand une exception est levée ou quand une assertion échoue. Il est conseillé d'utiliser `assert` pour tout ce qui doit être testé ou vérifié, avec un commentaire explicite en cas d'erreur.

Vos tests vont charger un modèle depuis le _MLFlow registry_ pour le tester, son nom et sa version sera défini par les variables d'environnement TEST_MODEL_NAME et TEST_MODEL_VERSION.

Vous devriez écrire une suite de test de complexité croissante, chaque test ne vérifiant qu'une seule caractéristique du modèle.

Voici les tests que vous devriez effectuer:
- Vérifier que le modèle fonctionne avec une entrée simple, codée en dur, et que le type de la sortie du modèle est correct.
- Vérifier que le modèle fonctionne avec des entrée unusuelles aussi (par exemple caractères spéciaux, entrée vide)
- Vérfiier que le modèle produit bien l'entrée attendue dans quelques cas évidents, codés en dur.
- Vérifier que l'_accuracy_ sur un jeu de test (défini en option par la variable d'environnement TEST_TEST_SET) est plus haute qu'un seuil donné.
- Vérifier que l'_accuracy_ sur un jeu de test (défini en option par la variable d'environnement TEST_TEST_SET) est plus haute que celle d'un modèle _baseline_ préalablement défini et enregistré dans MLFlow Registry (défini en option par les variables d'environnement TEST_BASELINE_MODEL and TEST_BASELINE_VERSION)

Les deux derniers tests seront executés en option si les variables d'environnement nécessaires sont définies.

Depuis vos notebook existant, définissez un modèle _baseline_, par exemple en prenant LogisticRegression et Tfidf avec la configuration par défaut. Enregistrez la dans le _MLFlow Registry_ avec un nom dédié, par exemple `sentiment-analyzer-baseline`.

## Executez les tests

Pour démarrer, executez la commande suivante:

```
TEST_MODEL_NAME=sentiment-analyzer TEST_MODEL_VERSION=5 TEST_FILE=./data/francais/test.csv pytest tests/
```

# Scripts Python

## Structurez votre projet

Les scripts python que vous allez écrire doivent pouvoir être installés pour être ensuite utilisés de n'importe ou en ligne de commande. Pour cela nous allons utiliser `setuptools`.

Ajoutez un fichier vide `__init__.py`  dans le repertoire `src` pour faire de votre projet un package python

Ecrivez ensuite un fichier `setup.py` à la racine de votre projet dont le contenu pourrait être:
```
from setuptools import setup, find_packages

setup(
    name='sentiment analyzer',
    version='0.1',
    packages=find_packages(),
    install_requires=[
        # Add your project dependencies here
    ],
    entry_points={
        'console_scripts': [
            'predict=src.predict_cli:main' #assuming predict is the command you want to install and src/predict_cli:main its source file
        ],
    },
)
```

Placez ensuite chacun de vos scripts python dans le folder `src`. Par exemple, avec le code ci-dessus, si vous placez dans `src` un programme `predict_cli` avec une fonction `main`, celle-ci sera executée, une fois votre package installé, quand vous appelerez la commande `predict` en ligne de commande. Ajoutez chacune de vos commande ainsi dans `console_script`.

Vous pouvez ensuite installer le package avec la commande suivante:
```
pip install -e .
```

## Prediction 

Implementez une commande `predict` qui calcule la _polarité_ d'un ou plusieurs messages à partir des modèles préalablement construits.

Utilisez `click` pour gérer les arguments. Voici les arguments que vous devirez utiliser:
- `--input_file`, le fichier d'entrée contenant les messages à traiter. Son format est similaires aux fichier déjà traités, c'est à dire que c'est un _csv_ contenant une colonne _review_ contenant les textes.
- `--output_file`, fichier de sortien, en csv
- `--text`, alternative à `input_file`, prends directement un texte unique en ligne de commande, calcule sa polarité et l'affiche sur al sortie standard.
- `--model_name`, nom du modèle utilisée tel qu'enregistré dans le _MLFlow registry_
- `--model_version`, version du modèle utilisée tel qu'enregistré dans le _MLFlow registry_
- `--mlflow_url`, URL du server MLFlow, valeur par défaut `http://localhost:5000`

Exactement un des arguments `--text` et `--input_file` doit être fourni.

Ecrivez la fonction `main` dans un fichier source `src/predict_cli.py` mais implémentez les différentes fonctionnalités dans une classe dédiée `ModelManager` dans un fichier `src/model_manager.py`. Vous vous en reservirez plus tard.

## Promotion

Créez un script `promote.sh` qui promeut un modèle dans le registre MLFlow.

Il doit prendre en paramètre ces arguments :

- `--model_name`, nom du modèle tel qu'il apparaît dans le registre MLFlow
- `--model_version`, version du modèle telle qu'elle apparaît dans le registre MLFlow
- `--status`, le statut auquel le modèle est promu: `staging`, `prod` ou `archive`

A noter que:
- Un modèle ne peut être promu que d'un statut au suivant.
- Un modèle ne peut être promu en `prod` que s'il passe les tests précédemment définis.

## Réapprentissage

Nous voulons pouvoir réentraîner automatiquement un modèle à partir de nouvelles données sans changer sa configuration d'entraînement.

Camme nous utilisons des pipelines _scikit-learn_, nous devons simplement charger le modèle et exécuter `fit` à nouveau. Une fois appris, le tag retrained doit être défini sur True. L'implémentation peut être effectuée dans la classe `ModelManager`, comme pour la commande `Prediction`.

Le script doit accepter les paramètres suivant :

- `--model_name`, tel qu'il apparaît dans le registre MLFlow
- `--model_version`, tel qu'il apparaît dans le registre MLFlow
- `--training-set`, le nouvel ensemble de données d'apprentissage à utiliser
- `--training-set-id`, identifiant optionnel de l'ensemble de données d'apprentissage.
- `--register-updated-model`, si défini, enregistre automatiquement le modèle dans le registre MLFlow, en tant que nouvelle version.

Ces tags doivent être définis :
- `retrained` qui doit être à `True`.
- `parent_version`, la version à partir de laquelle le modèle est reconstruit.
- `training_set_id`, si il a été fourni.
Si le modèle est enregistré dans _MLFLow_, les tags doivent être associés au modèle et au _run_. Sinon, uniquement au _run_.

