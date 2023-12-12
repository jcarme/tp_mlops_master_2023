# Seance 5 - Frontend, backend, orchestration

Dans cette séance nous allons passer du déploiement de notre webapp sur un unique conteneur docker au déploiement de l'ensemble de l'application sur un ensemble de conteneur orchestrés par _docker-compose_

## Logging

Pour pouvoir nous y retrouver dans le monitoring des differents conteneurs, nous allons implémenter le loggings des evenements dans notre webapp, et par la suite dans l'application _front_.

Pour cela, utilisez le module `loguru`. Celui-ci doit être ajouté dans les dépendances de votre webapp et importé. 

Modifiez votre webapp pour logguer, au moyen de `logger.info`, chaque appel à predict. En cas de levée exception, utilisez `logger.error`. Enfin, si vous avez besoin d'afficher des information de debuggage (par exemple les entrées/sorties de predict), utilisez `logger.debug`. C'est préférable à l'uilisation de `print`, notamment dans le cadre de l'utilisation de _Docker_.

## Application Frontend

### Developpement de l'application

Vous allez développer un _frontend_ pour votre application. Placez votre code dans le répertoire `src/frontend`

Pour cela, nous allons utiliser _streamlit_, une bibliothèque particulièrement adaptée pour permettre à des développeurs _backend_ de réaliser facilement, en Python, des applications _frontend_, notamment pour le prototypage.

Créez une application _streamlit_ avec:
- Un `st.header` contenant un titre
- Une `st.text_area` pour que l'utilisateur puisse entrer son texte
- Un `st.button` pour lancer la prédiction
- Un `st.success` pour afficher le résultat si tout s'est bien passé.

Pour appeler le service `predict` de votre webapp quand l'utilisateur clique sur le bouton, utilisez la bibliothèque `requests` et en particulier sa fonction `requests.post`.

Pour executer votre code, executez la fonction suivante depuis le repertoire `frontend`:

```
streamlit run app.py
```

### Creation du conteneur

Dans le répertoire `src/frontend` vous allez créer un `Dockerfile` pour votre application frontend. Construisez l'image, et executez la. Pensez à exposer le port de streamlit (par défaut 8501) et à le publier sur le host au moyen de l'argument `-p` de la commande `docker run`. 

## Deploiement de l'application avec docker-compose

Nous allons maintenant utiliser _docker-compose_ pour pouvoir déployer l'ensemble de l'application en une commande.

Créer à la racine de votre projet un fichier `docker-compose.yml` définissant deux services différents:
- Le service `webapp`. Ce service doit utiliser l'image déjà créée `sentiment-analyzer:<VERSION>`. La version (et eventuellement le nom de l'image) doivent être passés en variable d'environnement. Pas de section `build`.
- Le service `frontend`. La section correspondante doit contenir une parametre `build` faisant référence au repertoire source du `frontend` ainsi qu'un parametre `ports` décrivant la redirection d'un port de votre _host_ vers le port _streamlit_ du conteneur `frontend`.

Pour compiler l'ensemble des conteneur requis (dans ce cas, uniquement `frontend`):
```
docker-compose build
```

Pour executer l'application:
```
PREDICTON_CONTAINER="sentiment-analyzer:<YOUR_VERSION>" docker-compose up
```

## Stockage de l'activité dans MongoDB (optionnel)

### Mise en place de MongoDB

Nous allons stocker l'ensemble des entrées/sorties de l'utilisateur dans MongoDB pour analyse ultérieure. Au delà du stockage des entrée-sorties, Mongo peut plus généralement être utilisé pour stocker l'ensemble des métriques générées par l'application.

Vous allez ajouter dans votre `docker-compose.yml` un service `mongodb` utilisant l'image `mongodb:latest`. Pensez à spécifier les éléments suivants:
- Les variables d'environnement `MONGO_INITDB_ROOT_USERNAME` et `MONGO_INITDB_ROOT_PASSWORD` 
- Le volume `mongo-data` montée sur le répertoire `/data/db` du conteneur, permettant de stocker les données _Mongo_ dans un volume _docker_. C'est important car sans cela les données de Mongo sont seulement stockée dans le conteneur et disparaissent à sa destruction. 

### Modification de votre application

Vous allez maintenant modifier votre webapp pour stocker les entrées/sorties de la fonction `predict` à chaque appel dans _MongoDB_.

Créez ensuite un nouveau _endpoint_ `/history`, qui prend un paramètre `n` et renvoie les _n_ derniers éléments de l'historique.

Vous pouvez ensuite modifier votre _frontend_ pour y faire apparaitre deux onglets (`st.tabs`), l'un pour effectuer les requêtes (ce que vous avez déjà implémenté), et l'autre pour afficher l'historique.




