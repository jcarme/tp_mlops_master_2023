# Séance 2 - MLOps tracking

Nous allons maintenant logguer nos expérimentations et nos modèles dans MLFlow.

Vous allez interagir avec MLFlow via votre code Python et via l'interface Web. En suivant ce TP vous allez entrer dans MLFlow l'ensemble des informations pertinentes relatives à vos expérimentations. L'interface Web de MLFlow vous permet d'accéder à cette information de manière globale et centralisée, il vous revient de vous y connecter pour vous familiariser avec elle.

Pour l'ensemble du TP, vous pouvez utiliser la documentation de MLFlow disponible ici:
https://mlflow.org/docs/latest/index.html

## Mise en place du server mlflow

Vous allez utiliser votre propre instance de MLFlow, que vous ferez tourner dans votre propre compte.
* Installez MLFlow dans votre environnement _conda_:
```
pip install mlflow
```
* Créez un repertoire `mlflow` dans la racine de votre compte
* Depuis ce repertoire, dans un terminal dédié, démarrez:

```
mlflow server
```

Le server sera accessible à l'adresse http://localhost:5000

## MLFlow Tracking

### Logging des paramètres

Créer un notebook `model_design_2.ipynb` qui reprend la création d'un modèle (que vous choisierez) tel qu'effectuée dans le TP précédent, en y ajoutant les éléments suivants:
* Connection au server MLFlow
* Activation de l'_autolog_ de MLFlow.sklearn
* Création de l'_experiment_ MLFlow
* Creation du _run_ MLFlow
* En fin de notebook, fermeture du run MLFlow

Une fois l'autolog effectué, executez votre notebook (en laissant pour l'instant de côté la partie _hyperopt_), consulter l'interface MLFlow et regardez ce qui a été loggué.

Le point clef est la **reproductibilité**. Nous voulons sauvegarder dans l _run_ l'ensemble des éléments qui pourront nous permettre de reproduire l'expérience.

Voici la liste des éléments à sauvegarder:
  * Identification du code source, branche, commit sous forme de tags
  * Identification du dataset
  * Identification du préprocessing et des algorithmes utilisés sous forme de tags et/ou parametres
  * Valeur des paramètres des algorithmes, sous forme de paramètres MLFlow
  * Un description en texte libre

Pour chacun de ces éléments, vérifiez si il est déjà sauvegardé par _autolog_, et dans le cas contraire, ajouter un logging manuel.

Dans le cas du nom du code source, du tag et du commit, _autolog_ ne fonctionne pas correctement du fait de l'utilisation d'un notebook. Stockez les donc manuellement sous forme de tags `mlflow.source.name`, `mlflow.source.git.commit` et `mlflow.source.git.commit`.

### Logging des métriques

Consultez les métriques logguées par _autolog_ lors de l'entrainement. __Qu'en pensez-vous ? Ces métriques sont-elles satisfaisantes pour évaluer la qualité de votre modèle?__

Ajoutez au moins une métrique pertinente d'évaluation de votre modèle.

### Consultation du server MLFlow

Prenez le temps maintenant de bien analyser l'ensemble des informations stockées dans MLFlow.
Depuis la page principale, une fois votre expérience choisie, allez dans l'onglet _table_, et ajoutez les informations manquantes, en particulier _source_, _version_, ainsi qu'une ou deux métrique bien choisie.

Vous pouvez ensuite cliquer sur un run donné et analyser l'ensemble des paramètres et métriques associées. Vérifiez que toutes les informations souhaitée sont bien là.

## MLFLow Registry

Vous remarquerez que l'_autolog_ sauvegarde le modèle, et qu'il est accessible depuis chacun des runs MLFlow, dans l'interface MLFlow.

Il n'est cependant pas _enregistré_ au sens du _registry_ MLFlow. L'idée est de sauvgarder dans le registre les modèles qu'on considère comme satisfaisant. Il sera alors accessible dans depuis l'onglet _Models_ de l'interface MLFlow.

Enregistrez votre modèle dans MLFlow programmatiquement (il est également possible de le faire depuis l'interface MLFlow mais ça n'est pas l'objet de ce TP). Pour cela, utilisez la fonction `mlflow.sklearn.log_model`

Associez également une description et un ou plusieurs _tags_ à votre modèle. Ces fonctionnalité ne sont possible qu'en ayant instancié un objet `MLFlowClient`.

Allez ensuite dans votre interface MLFlow pour visualiser votre modèle enriegistré. Vous noterez que les paramètres et les métriques ne sont pas directement associés au modèle. Elle le sont au _run_ qui a créé le modèle. Vous pouvez donc les retrouver en cliquant, depuis la page de la version choisie de votre modèle, sur `source_run` .


## Conception d'une fonction d'expérimentation

Pour mener des expériences de manière plus systématique, nous allons encapsuler la conception des modèles.

Nous allons écrire dans un nouveau notebook `model_design_3.ipynb` une fonction qui créé un modèle et loggue l'ensemble des éléments demandés dans MLFlow:

    def build_model(
        dataset,
        pipeline,
        model_name,
        mlflow_run_tags = None,
        mlflow_run_parameters = None,
        mlflow_run_description = None,
        mlflow_model_tags = None,
        mlflow_model_description = None
    ):
    """
    Build a sentiment analysis model, print the evaluation result and store everything to MLFlow
    @param: dataset: pandas dataframe containing the input training set
    @param: pipeline: scikit-learn pipeline that will be applied to the input data
    @param: model_name: name of the model as it will be stored in MLFlow
    @param: mlflow_run_tags: dict of tags that will be stored in the MLFlow run
    @param: mlflow_run_parameters: dict of parameters that will be stored in the MLFlow run
    @param: mlflow_run_description: textual description of the run
    @param: mlflow_model_tags: dict of tags that will be stored in the MLFlow regietered model
    @param: mlflow_model_description: textual description of the model    
    @return: the ModelInfo of the model generated by MLFlow 

La création d'un modèle peut alors ensuite se ramener à :
* La conception d'un pipeline
* La définition d'un _experiment_ MLFlow et des informations MLFlow spécifique à l'experience
* L'appel à la fonction `build_model`

### Experimentations

Vous pouvez maintenant mener différentes expérimentations qui seront stockées dans votre server MLFlow. Reprenez la section "Expérimentation de différents modèles et hyperparamètres" du TP précédent, et faite de même avec votre nouvelle fonction, qui loggue l'ensemble des données dans MLFlow.

Chacune de ces expérimentations doit être effectuée par une cellule de votre notebook précédée d'un paragraphe explicatif et suivi des résultats et d'une éventuelle conclusion. 

Chaque expérimentation doit être logguée dans un _experiment_ indépendant de MLFLow, avec son propre nom. Si par contr vous changez juste un ou plusieurs paramètres ou vous relancez plusieurs fois la même experimentation pour une raison ou une autre, faites plusieurs runs dans la même expérimentation.

Consultez votre server MLFlow et analysez en détail l'ensemble des informations qui y sont logguées.
En utilisation les fonctionnalités de l'interface, adaptez la tableau de bord à vos besoins, supprimez le colonnes et diagrammes inutiles et ajoutez ceux correspondant aux métriques que vous avez ajoutées.

## Optimisation des hyperparametres 

### Adaptation de la fonction de construiction des modèles

Créez une fonction `build_optimized_model` la capacité de trouver les hyper parametres optimaux au moyen de la bibliothèque `hyperopt`. Pour plus de détails, consultez la section "Optimisation des hyper-paramètres" du TP précédent.

En plus des paramètres de la fonction `build_model`, votre fonction `build_optimized_model` devra recevoir l'ensemble de validation et l'espace d'optimisation tel que demandé par la fonction `hyperopt.fmin`. Les paramètres 

La fonction `objective` qu'_hyperopt_ va minimiser doit être une fonction calculée sur __l'ensemble de validation__. Plus elle est faible, meilleure est le modèle, typiquement, vous pouvez renvoyer `1-accuracy`.

Enfin, une fois le paramétrage optimal trouvé, le modèle final doit être reconstruit (ce qu'_hyperopt_ ne fait pas), loggué, en enregistré dans le _mlflow registry_.

### Logging MLFlow

Adaptez le logging MLFlow en conséquence. Créez un nouveau run à chaque étape, c'est à dire au sein de la fonction `objective`.

**Selon vos, quelle métrique est pertinente à logguer pour chaque étape de l'optimisation?**

Comme vous allez générer ainsi un nombre important de runs, il est important d'utiliser des tags pour vous y retrouver dans l'interface MLFlow. Par exemple, vous pouvez utiliser les tags suivants:
- `hyperopt_candidate=True` pour les modèles générés par _hyperopt_ lors de sa recherche
- `hyperopt_selected=True` pour les modèles finaux sélectionnés par _hyperopt_.

Une fois dans l'interface MLFlow, il sera alors facile de visiualiser uniquement les runs que l'on souhaite, par exemple avec le filtre:
`hyperopt_candidate!=True or hyperopt_selected==True` pour n'afficher que les runs "manuels" et les runs optimaux de hyperopt, sans afficher tous les résultats intermédiaire de hyperopt.

Une fois que vous avez choisi un modèle entièrement satisfaisant, vous pouvez le passer en mode "staging" dans le repository MLFlow, depuis votre notebook.

### Amélioration de la fonction d'optimisation (optionel)

Vous remarquerez que la fonction d'optimisation recalcule l'ensemble du pipeline a chaque étape, même si les paramètres qu'on cherche à optimiser ne concernent que la deuxième étape (le _train_). Modifiez votre code de manière à ce que les étapes précédant l'étape à optimiser ne soient pas recalculées. 

Cette fois-ci, votre fonction prendra en paramètre: 
- Le nom de l'étape à optimiser dans le pipeline scikit-learn (par exemple `logreg`)
- L'espace des paramètres, cette fois-ci non préfixé (par exemple `C` et non `logreg__C`)


