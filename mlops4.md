# Séance 4 - Deployement

## Créer une webapp "sentiment-analyzer"

### Chargement préalable du modèle

Pour simplifier, dans cette première partie, notre Webapp chargera un modèle stocké localement. Récupérez le dans MLFlow au moyen du script `get_mlflow_model.py` suivant:

```
import click
import mlflow
import mlflow.sklearn

@click.command()
@click.option("--mlflow_server_uri")
@click.option("--model_name")
@click.option("--model_version")
@click.option("--target_path")

def main(mlflow_server_uri, model_name, model_version, target_path):
    mlflow.set_tracking_uri(mlflow_server_uri)
    model = mlflow.sklearn.load_model(model_uri=f'models:/{model_name}/{model_version}')
    mlflow.sklearn.save_model(model, target_path)

if __name__ == "__main__":
    main()
```

Adaptez les valeurs de paramètres à votre cas, executez ce script. Pour `target_path`, choisissez un stockage temporaire, par exemple `/tmp/sentiment-analyzer-model`, et sauvegardez le dans la variable d'environnement `SENTIMENT_ANALYZER_MODEL_PATH`.

### Version de base

Vous allez créer une webapp `sentiment-analyzer` avec FastAPI, qui pourra être interrogée en REST et effectuer une prédiction.

Vous aurez besoin d'ajouter dans vos requirements les modules fastapi et uvicorn.

Dans votre repertoire racine, créez un repertoire `webapp`. Dans ce répertoire, copiez le fichier `get_mlflow_model.py` et créez un fichier source `app.py` qui sera le code source de votre webapp.

Votre webapp doit implémenter:
- Le chargement du modèle stocké dans `SENTIMENT_ANALYZER_MODEL_PATH` (qui sera chargé depuis une variable d'environnement)
- Un endpoint `/predict`, de type POST, qui:
  - Prend en entrée une structure comme: `{"reviews":["c'est un bon film","c'est un mauvais film"]}`
  - Renvoie en sortie `{"sentiments":["positif","negatif"]}`

La structure générale de `app.py` doit être:
```
from fastapi import FastAPI
from pydantic import BaseModel
<...>

app=FastAPI()

class PredictInput(BaseModel):
    reviews: list[str]

<init code>

@app.post(<your_endpoint>)
def <your_endpoint>(input:PredictInput):
    <your_endpoint_code>
```

Attention au typage de ce que votre modèle va retourner. Il s'agit d'un `numpy.Array` et non d'une liste.

### Execution et tests

Pour lancer le server, executez:
```
uvicorn app:app --host 0.0.0.0
```

Pour tester votre code, vous pouvez accéder à sa doc qui est automatiquement accessible à l'adresse suivante: `http://localhost:8000`
La documentation vous permet de tester directement les endpoints via le bouton "try it out".

### Documentation

Pour que la documentation générée en ligne soit la plus complète possible, vous pouvez:
- Ajouter les arguments `title`, `version` et `description` à l'instanciation de `FastAPI`
- Ajouter un paramètre `summary` à votre décorateur `@app.post`
- Commentez votre fonction predict.

## Déployez votre webapp dans docker.

Nous allons créer un conteneur docker qui contiendra un et un seul modèle, non modifiable. L'idée est de créer un conteneur pour chaque modèle déployé.

### Installer Docker

Selon votre environnement, Docker sera déjà installé ou non. Sous windows ou WSL2, vous pouvez installer Docker Desktop. Si vous utilisez WSL2, sélectionnez l'option correspondante dans les paramètres Docker Desktop.

Pour tester votre installation, executez:
```
docker run hello-world
```

### Création de l'image docker

Dans votre repertoire `webapp`, vous allez créer un fichier `Dockerfile` qui contiendra l'ensemble de instruction de construction de votre conteneur _Docker_.

Le Dockerfile doit implémenter les étapes suivantes:
- Partir d'une image légère générique, par exemple `python:3.9-slim`
- Choisir un repertoire de travail dans le conteneur, par exemple `/webapp`
- Installer les packages requis, hors modèle: `mlflow`, `fastapi`, `uvicorn`.
- Copier les fichiers source requis.
- Executer le script du récupération du modèle depuis votre server MLFlow. Ce script utilisera des arguments de compilation MLFLOW_SERVER_URI, MODEL_NAME et MODEL_VERSION. Le modèle sera installé dans un repertoire local du conteneur, par exemple `/model`
- Installer les requirements du modèles, qui se trouveront dans `/model/requirements.txt`
- Exposer le port 8000, le port d'accès à votre Webapp
- Executer la commande `uvicorn` de démarrage de votre Webapp.

Vous pouvez ensuite compiler votre dockerfile avec la commande (que vous pourrez mettre dans un script):
```
docker build 
    -t sentiment-analyzer:<TAG>
    --build-arg <PARAMETER_1>=<VALUE_1>
    --build-arg <PARAMETER_2>=<VALUE_2>
    .
```
Pour la valeur de <TAG>, je suggère d'utiliser la convention <CODE_VERSION>-<MODEL_NAME>-<MODEL_VERSION>, qui permet de voir instantanément dan sla liste des images Docker de quelle version du code et du modèle il s'agit.
Pour les <PARAMETER_N>, il s'agit des paramètres utilisés dans votre Dockerfile.

__Attention!__, les commandes executées dans le dockerfile le sont dans le conteneur qui a sa propre adresse réseau, elles n'ont pas directement accès à votre `localhost`. Pour l'adresse de votre server MLFlow, vous devez utilisez l'adresse du `host` et non `localhost`. Pour cela, vous pouvez utiliser l'adresse speciale `host.docker.internal`. Votre server MLFLow aura donc l'adresse `http://host.docker.internal:5000`

### Execution de votre conteneur

Il vous reste à executer votre conteneur et vérifier son bon fonctionnement.

Pour cela, executez la commande:
```
docker run 
    -p 8080:8000 
    --name <CONTAINER_NAME>
    <IMAGE_NAME>
```

Le `-p 8001:8000` vous permet de rediriger le port 8000 de votre conteneur (celui sur lequel on accède à la webapp) vers le port 8001 de votre host. Vous pouvez ensuite accéder à votre webapp dans le conteneur depuis votre navigateur à l'adresse `http://localhost:8001`

Vous pouvez également, pour débugger ou par curiosité, executer un shell dans votre conteneur ainsi:
```
docker exec -it <CONTAINER_NAME> sh
```

## Améliorations (optionnel)

### Model details

Vous pouvez modifier votre script `get_mlflow_model.py` pour qu'il charge également les informations associées au modèles, les `model details`. Pour cela vous pouvez utiliser `mlflow.get_model_version`, et stocker le résultat en json dans le conteneur au même titre que le modèle.

Vous pouvez alors 
- Ajouter un endpoint `get_details` qui renvoie les détail du modèle utilisé par le conteneur.
- Ajouter un endpoint `get_stage` qui renvoie le _stage level_, à savoir `None`, `Staging`, `Production` ou `Archive`.







