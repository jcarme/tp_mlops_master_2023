# Séance 6 - Huggingface, Github, Docker Hub

Au cours de cette séance nous allons exaprimenter divers plateforme publiques de MLOps.

## Huggingface

### Création d'un compte Huggingface

Huggingface est la plus grande plateforme publique de gestion et de partage de modèles ML:

https://huggingface.co/

Nous allons publier notre modèle dans Huggingface. Une fois publié, le modèle sera disponible publiquement en téléchargement manuel ou automatisé. A noter que Huggingface permet également de rendre les modèles privé, et permet à une entreprise, via des offres dédiée, d'être utilisée comme une plateforme privée MLOps de gestion des modèles.

Commencez par vous créer un compte Huggingface, ainsi que 2 tokens, l'un pour la lecture, l'autre pour l'ecriture.

### Script de déploiement d'un modèle Huggingface

Huggingface propose une bibliothèque nommée `skops` qui est spécifiquement dédiée à l'intégration des modèles _scikit_learn_ dans des environnements de production, et en particulier Huggingface.

Vous allez ajouter dans votre package `sentiment-analyzer` une commande `hf_export` qui prend en paramètre un modèle MLFlow et l'exporte sur Huggingface. Huggingface ne gére pas directement le format MLFlow, mais il gère le fichier _model.pkl_ qui est situé dans le dossier du modèle MLFlow.

La commande prendra en arguments:
- `--mlflow_url`, URL du server MLFlow, valeur par défaut `http://localhost:5000`
- `--model_name`, nom du modèle utilisée tel qu'enregistré dans le _MLFlow registry_
- `--model_version`, version du modèle utilisée tel qu'enregistré dans le _MLFlow registry_
- `--data_file`, the data file used to learn the model, requested by HuggingFace
- `--hf_id`, votre identifiant HuggingFace
- `--hf_token`, votre token d'accès en écriture à Huggingface.

Pour cela, vous devez:
- Récupérer le modèle depuis MLFlow et le stocker sur un stockage temporaire local, par exemple `/tmp/model`
- Charger le jeu de données dans une _datatable_ Pandas
- Charger le richier `/tmp/model/requirements.txt` sous forme d'une liste (utilisez `file.readlines()`)
- Générez le nom pour votre repository hugginface ainsi:
```
<id_huggingface>/<model_name>-<version>
```
- Utiliser ensuite les commandes suivantes de la bibliothèque `skop` suivantes:
  - `hub_utils.init` pour créer votre modèle Huggingface.
  - `hub_utils.push` pour l'uploader vers Huggingface

A chaque modèle Huggingface peut être associée une _model card_, c'est à dire un fichier de description de ce modèle. Ca n'est pas nécessaire dans le cadre de ce TP, mais il serait pertinent d'extraire toutes les informations associée au modèle dans MLFlow et de générer à partir de celle-ci une _model card_. `skops` propose des fonctionnalités spécifiquement dédiées à la génération de _model cards_

Une fois votre script créez, utilisez le pour charger dans huggingface l'un de vos modèles MLFlow.

## Adaptation de la Webapp

Malheureusement la WebApp qui a été conçue pour fonctionner sur des modèles MLFlow doit subir quelques adaptations pour être en capacité de fonctionner avec les modèles _pkl_ gérés par HuggingFace.

Pour simplifier nous allons créer une autre version de la Webapp:
- Copiez le repertoire `webapp` dans un autre répertoire `webapp.hf`
- Le Dockerfile doit être adapté ainsi, pour charger directement le modèle de Huggingface à la place de MLFlow:
```
FROM python:3.9-slim

WORKDIR /webapp
RUN pip install mlflow fastapi uvicorn loguru pymongo skops

ARG HF_ID
ARG MODEL_NAME
ARG MODEL_VERSION

RUN python -c "from skops import hub_utils; hub_utils.download(repo_id='${HF_ID}/${MODEL_NAME}-${MODEL_VERSION}', dst='/model')"
COPY app.py ./
EXPOSE 80
CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "80"]
```
- Adaptez `app.py` pour charger le modèle en utilisant `joblib.load` à la place de `mlflow.sklearn.load_model` pour charger directement le fichier `/model/model.pkl`

Vous pouvez maintenant construire la nouvelle image en utilisant `docker build`, de la même façon que dans la séance précédente. Donnez lui un nom différent comme `sentiment_analyzer-webapp-hf`. Vous pouvez ensuite la tester en déployant votre application `docker-compose` avec cette nouvelle image.

## Docker Hub

Le Docker Hub est la plateforme publique de stockage des images docker. Une fois uploadées sur la plateforme, les images Docker sont directement accessibles et utilisables par tout le monde. Par exemple, si l'utilisateur `jcarme` créé et pousse sur docker hub une image nommée `jcarme/my_app:1.0`, n'importe qui pourra depuis n'importe ou executer son application avec la commande suivante:

```
docker run jcarme/my_app:1.0
```

Vous allez vous créer un compte sur Docker Hub et générer un token d'accès lecture/ecriture.

## Déploiement Continu avec GitHub

Nous allons maintenant mettre en place le déploiement continu dans GitHub. Le principe est le suivant: à chaque nouvelle version de l'application poussée dans la branche `main`, l'ensemble des images _docker_ seront régénérées, ce qui inclue la récupération du modèle dans HuggingFace, puis poussées ver le Docker Hub de manière à être mise à la disposition des utilisateurs.

Pour cela, nous allons définit une _GitHub Action_, c'est-à-dire un script qui sera executé sur un évenement particulier, dans notre cas sur le push de nouveau code, ou manuellement en cas de mise à jour du modèle sur HuggingFace.

### Définitions des variables de configuration

Dans notre projet GitHub, nous allons définir un certain nombre de paramètres qui seront nécessaires à l'execution de notre action.
Cela se fait au moyen des _variables_ et des _secrets_. Les _variables_ sont publiques, les _secrets_ privés. Le réglage de ces paramètres se trouve dans _settings/secrets and variables/actions_

Vous devez définir les paramètres suivants:
- DOCKER_HUB_USERNAME
- DOCKER_HUB_PASSWORD
- HUGGINGFACE_ID
- MODEL_NAME
- MODEL_VERSION
- CODE_VERSION

Dans le fichier de configuration qui va suivre, pour accéder ensuite au _variables_, vous devez utiliser la syntaxe `${{ vars.MY_VAR }}`, et pour accéder à un secret, la syntaxe `${{ secrets.MY_SECRET}}`

### Définition de la _github action_

La _github action_ se défini au moyen d'un fichier _yml_, par exemple `docker-build.yml` qui doit se trouver dans un repertoire `.github/workflows` à la racine de votre projet. 

Le fichier doit contenir une seciont `on` qui défini les evenements déclenchant l'action et une section jobs qui défini l'action elle-même.

Voici la structure du fichier que vous devez créer:

```
name: Docker Images Build

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build-and-push:
    runs-on: ubuntu-latest

    steps:
    - name: Check Out Repo
      uses: actions/checkout@v2

    - name: Log in to Docker Hub
      uses: docker/login-action@v1
      with:
        username: <...>
        password: <...>

    - name: Build and Push Docker image 1
      uses: docker/build-push-action@v2
      with:
        context: <path_to_your_image_root>
        file: <path_to_your_dockerfile>
        push: true
        tags: <docker_hub_username>/<image_name>:<image_version>
        build-args: |
          <BUILD_PARAMETER_1>=<value1>
          <BUILD_PARAMETER_1>=<value2>

    - name: Build and Push Docker image 2
      <...>
```

Une fois votre fichier créez, poussez le sur votre repo github et l'action va se déclencher automatiquement. Vous pourrez suivre son déroulement et, si besoin, debugger, et allant dans la section _Actions_ et en cliquant sur le job correpondant.

### Récupérez vos images

Maintenant que vos images sont uploadées dans Docker Hub, elles sont directement accessibles. 

Pour tester cela, vous pouver créer hors de votre projet un dossier vide dans lequel vous pouvez placer simplement un fichier `docker-compose.yml` ne contenant aucune instruction de `build`, uniquement des référence vers les images à télécharger:

```
version: '3.8'

services:
  webapp:
    image: <docker-hub-user>/<webapp-image>:<version>

  frontend:
    image: <docker-hub-user>/<frontend-image>:<version>
    ports:
      - "8501:8501"
    environment:
      - WEBAPP_URL=http://webapp:8000
    depends_on:
      - webapp

  mongodb:
    image: mongo:latest
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: example
    volumes:
      - mongo-data:/data/db


volumes:
  mongo-data:
```

En executant alors `docker-compose up`, vos images seront téléchargées depuis Docker Hub et executées. L'applications devrai alors être accessible à http://localhost:8000.

### Important! Supprimez vos images de Docker Hub

Une fois le TP terminé, Merci de vous logguer sur le site de Docker Hub et de supprimer vos images.
