

# Séance 1

## Mise en place de l'environnement

### Git

Le versionning de code est indispensable à toute démarche DevOps/MLOps.

* Si vous n'en disposez pas, créez vous un compte sur Github.
* Allez dans _Settings/Developper Settings_ pour vous créer un token d'accès.
* Créer un repository privé sur Git hub dédié à votre projet MLOps, nommez le _mlops_project_. Dans la page "Create a new repository", précisez que vous souhaitez que celui-ci soit privé.
* Ajoutez **jcarme** à la liste des membres de votre projet.
* **A la fin de chaque séance (a minima), poussez la version en cours de votre projet.**

Une fois votre repository créé, récupérez le en local sur votre machine et créez la structure de fichier suivante:

    |-- README.md
    |-- requirements.txt
    |-- notebooks
        |-- model_design.ipynb

Le rôle de ces fichiers est le suivant:
* _README.md_ contient la documentation, en anglais. Il doit permettre à n'importe quel nouvel utilisateur de pouvoir comprendre et utilise votre projet de manière immédiate. Maintenez-le à jour tout au long de l'évolutiond e votre projet.
* _notebooks_ est un dossier qui contient l'ensemble des notebooks que vous allez créer.
* _requirements.txt_ contient l'ensemble des package nécessaires au fonctionnement de votre projet, un par ligne, de préférence versionnés.

L'intérêt de _requirements.txt_ est qu'il permet l'installation, dans l'environnement en cours, de l'ensemble de packages requis en une commande:
```
pip install -r requirements.txt
```

D'autres fichiers et dossiers seront ajoutés au fur et à mesure à cette structure initiale.

### Conda

Vérifiez que la commande `conda` est disponible.

Créez un nouvel environement _conda_, appelez le _mlops_:
```
conda create --name mlops python=3.11
conda activate mlops
```

A chaque nouveau package python dont vous aurez besoin dans votre projet, ajoutez le dans le _requirements.txt_. Vous pou


## Récupération des données

Téléchargez le jeu de données à cette adresse: https://drive.google.com/file/d/1i2T30NH_PCwDJD2si7Rp_ZMPxPYBSatb/view?usp=sharing

Décompressez l'archive et récupérez les 3 jeux de données: _train_, _test_ et _validate_

## Analyse des données

Vote outil principal pour l'ensemble de travaux d'analyse, et de modellisation sera [Scikit-Learn](https://scikit-learn.org/).


### Analyse exploratoire 

D'une manière générale, il est conseillé de commencer d'effectuer une première analyse exploratoire des données avant tout travail de modélisation. Ce n'est pas l'objet essentiel de ce TP mais vous pouvez cependant rapidement vérifier, par exemple, l'absence de valeurs manquantes et la distribution de valeurs du score, et déjà vous familiariser avec le contenu textuel.

Ces travaux doivent être effectués dans un _notebook_ dédié que vous pouvez nommer _exploratory_analysis_ et que vous stockerez dans le dossier _notebooks_. Utilisez la bibliothèque _pandas_ pour charger le fichier _csv_ en mémoire et l'analyser.

## Prétraitement des données 

Avant de pouvoir être traitées par un algorithme de machine learning classique, les textes doivent être transformés en _features_. Pour cela, vous allez utiliser un algorithme de la bibliothèque _scikit-learn_ et plus particulièrement le module _sklearn.feature_extraction_. Consultez la documentation et faite vous une idée des différents algorithmes que vous pouvez utiliser, ainsi que de leur paramétrage:

https://scikit-learn.org/stable/modules/classes.html#module-sklearn.feature_extraction

Dans le notebook _model_design.ipynb_, écrivez une première section "Preprocessing" dans laquelle vous implémentez cette phase. N'oubliez pas la suppression des _stop words_, et pensez au fait que vos textes sont en Français. La bibliothèque _spacy_ peut vous fournir les _stop_words_ français.

D'une manière générale, dans vos notebooks, alternez description en markdown et implémentation en python.

## Conception du modèle

Une fois la phase de prétraitement effectuée vous pouvez construire votre modèle. Dans un premier temps, vous allez utiliser un modèle simple qui constituera une _baseline_, comme par exemple _sklearn.linear_model.LogisticRegression_

Effectuez l'apprentissage sur l'ensemble d'entrainement et l'évaluation de votre modèle sur l'ensemble de test. Pour les métriques d'évaluation, utilisez les fonctionnalités offertes par _sklearn.metrics_. Consultez la documentation.

**Choix des métriques**: Que signifient _precision_, _recall_, _accuracy_  dans le contexte de ce TP ? Qu'en concluez vous sur la métrique à privilégier ?

## Création du pipeline scikit-learn

Créez un Pipeline scikit-learn contenant à la fois le prétraitement et la création du modèle. Appliquez les routines d'apprentissage et d'évaluation directement sur ce Pipeline.

## Expérimentation de différents modèles et hyperparamètres

Testez différents paramétrages de l'algorithme LogisticRegression. Vous pouvez en particulier jouer avec les paramêtres _C_ et _penalty_.
Testez différents algorithmes d'apprentissage, par exemple _SVC_ et _MLP_, et cherchez à tester différents hyperparamètres. Par exemple, vous pouvez tester plusieurs structure de réseau en _MLP_.

Chaque expérimentation soit être précédée d'un paragraphe explicatif. Une description textuelle courte doit également être sauvegardée dans MLFlow.


## Optimisation des hyper parametres

Si il vous reste du temps, vous pouvez expérimenter l'optimisation des hyperparametres au moyen de la bibliothèque _hyperopt_. Sa documentation est disponible ici:

https://github.com/hyperopt/hyperopt

Ne faites pas d'effort particulier sur le reporting des résultats, car ce sera l'objet de travaux ulterieurs avec la bibliothèque MLFlow.

Quelques conseils sur l'utilisation de _hyperopt_:
* Vous devez utiliser la fonction `hyperopt.fmin` avec comme arguments obligatoires:
  * `fn`, la fonction à minimiser, que nous appeleront `objective`. Cette fonction doit effectuer un entrainement et renvoyer le résultat de l'évaluation sur le _validation set_. **Attention** il faut renvoyer une valeur à minimiser, vous pouvez par exemple prendre `1-accuracy`
  * `space`, l'espace de recherche des hyperparamètres. Consultez la documentation de fmin (https://github.com/hyperopt/hyperopt/wiki/FMin) pour voir comment le définir. Dans le cas d'un _pipeline_ _scikit-learn_ les    paramètres sont définit ainsi: le paramètre `C` de l'étape du pipeline nommée `logreg` est nommé `logreg__C`
  * `max_evals`, le nombre maximal d'itératons. Prenez une valeur très basse pour tester, par exemple 10, car sinon le temps de calcul risque d'être trop long pour pouvoir être effectué pendant la durée du TP. Vous pourrez ensuite le laisser tourner avec une valeur beaucoup plus grande, si vous en avez la possibilité.












