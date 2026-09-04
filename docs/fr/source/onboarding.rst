Onboarding plateforme incubation
=============================

Ressources disponibles
----------------------

Les ressources mises à disposition pour le centre d'incubation dédiés à
l'exécution des chaines sont les suivantes, **elles sont mutualisés:**

* 4 Noeuds DEV1-XL (4CPU, 12GB)
* 220Gi de stockage pour les fichiers temporaires lors de l'exécution

Les ressources disponibles pourrons être amenés à évoluer.

.. warning::
   Il est important de bien spécifier dans votre OGC package
   (CWL) les ressources nécessaires au lancement de votre chaîne, en
   particulier les limites CPU et mémoire

   Pour plus d'information voir: :ref:`hints`

.. _hints_requirement:

Pré requis
----------

* Un workflow CWL de une ou plusieurs tâches utilisant des conteneurs et ne dépendant de aucun partage POSIX.

   * Quelques briques utiles: :doc:`cwl_quick_start`

* Les images de conteneurs doivent être accessibles depuis le cluster de la plateforme d'incubation, trois choix sont possible:

      * Ajouter le groupe: **platform-incub-token** en accès RO sur votre registry Artifactory
      * Pousser votre image sur: **platform-incub-docker-prod-local/cwl-processing** (droit ajoutés à la demande).
      * Rendre accessible **publiquement** votre image

* Posséder les données d'entrées pour le lancement de la chaîne (upload dans bucket de dev, EODAG…)
      * Le bucket peut être hebergé sur n'importe quel provider
* Un compte Gitlab CNES
* Respecter la guideline suivante : https://processing-chain-guidelines.readthedocs.io/en/latest/

Workflow
--------

Vous trouverez ci-dessous un schéma macro des étapes d'utilisation du centre expert. 

Les points clés sont les suivant :

* Votre bucket de développement est uniquement utilisé pour les données d'entrée de votre chaîne ainsi que par le code effectuant le benchmark

   * La procédure d'import d'un secret passée en variable d'environnement de votre chaîne sera expliquée plus bas

* Les données en sortie de votre chaîne sont stockés dans le bucket dédié à Antflow, il est possible pour vous de les explorer avec le STAC browser, ainsi que les télécharger.
* Votre document CWL peut être stocké dans un dépôt dédié ou bien dans le dépôt ou se trouve votre code.

.. thumbnail:: images/centre_expert_workflow.png

Création d'un bucket S3 dédié au développent (Accès au réseau CNES nécéssaire)
--------------------------------------------

Ce bucket va vous permettre de stocker les fichiers suivants:

* N'importe quel fichier d'entrée de votre chaine, récupération à l'exécution, soit par une étape dédié (voir ici :doc:`CWL Quick Start`) ou par votre chaîne en elle même.
* Le fichier de configuration du benchmark (voir suite)

.. warning::
   Vous devez être en mesure d'accèder au réseau bureautique CNES pour utiliser la plateforme de self service du CNES

Se rendre sur la plateforme de self service cloud:
https://squest.admin.cloud.cnes.fr

Se connecter avec son compte IPA.

Créer un compte de service avec l'offre "Service Account (Scaleway)", vous allez obtenir une clé d'accès ansi qu'une clé secrete par email

Sélectionner l'offre "S3 personnalisé (Scaleway)" 

Utiliser le compte de service créer précédement pour ajouter vos droits au bucket

Par exemple: **application_id:<id> | s3:* | <bucket_name>, <bucket_name>/*** pour un accès complet


.. warning::
   Entrer la ligne suivante dans la boite de texte "custom_policy_lines": **application_id:ef62a65b-b3c9-4e3f-8a1f-a6cf6cd23cf2 | s3:ListBucket, s3:GetObject | <bucket_name>, <bucket_name>/***
   
   Cela va permettre au script de bench d'avoir un accès RO à votre
   bucket et pouvoir récupérer le fichier de configuration (voir plus
   bas)

   Suivre le guide ci-dessous pour plus de détails

.. thumbnail:: images/s3_squest.gif

Lancement de votre workflow CWL
-------------------------------

Pour la configuration de votre projet Gitlab:
https://antflow-iac.geo-platformlab.fr/ui/#/docs/references/project-setup

.. warning::
   Quelques minutes (2-3) peuvent être nécéssaire avant que votre processus se lance, 
   cela est dû à la création à la volée des noeuds de calcul en charge 
   de l'éxécution de votre workflow

Ajout des variable d'environnements
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Si vous souhaitez ajouter des variables d'environnement lors de
l'exécution:
https://antflow-iac.geo-platformlab.fr/ui/#/docs/references/env-variables

.. warning::

   Toutes les variables doivent être préfixés par :
   “CWL\_\_”

   Les variables doivent être en mode “visible” et “protégés”

.. thumbnail:: images/CWL_project_pipeline_var.png

Lancement du benchmark de compliance
------------------------------------
.. warning::
   **Vous avez 3 jours** pour lancer le benchmark d'un
   workflow exécuté, passé ce délais, les métriques serons supprimées.
   Il vous faudra donc lancer votre workflow à nouveau.

Configuration
~~~~~~~~~~~~~

Un seul fichier de configuration est nécessaire pour le benchmark, voici
un exemple:

.. code:: python

   {
       "registry.gitlab.com/my_registry/project/image:tag" {
           "link": "https://gitlab.orfeo-toolbox.org/project/blaba",
           "token": "" <-- projet public
       },
       "artifactory.cnes.fr/my_registry/project/image:tag": {
           "link": "https://gitlab.cnes.fr/Theia/surf-water",
           "token": "xxx" <-- projet privé
       }
   }

Afin d'avoir les métriques de profiling, l'outil de benchmark doit
pouvoir accéder au code exécuté dans le conteneur de votre chaîne, pour
cela il vous faut spécifier **l'image de conteneur et le repo
correspondant.**

.. warning::
   Le nom de l'image doit correspondre à celui présent dans
   votre workflow CWL

Un accès en RO avec un Access Token Gitlab est suffisant si votre repo
est protégé.

**Si vous ne souhaitez pas avoir le profiling,** ne renseignez pas de
nom de bucket/chemin vers le fichier de conf dans le formulaire du
benchmark

Lancement
~~~~~~~~~

Rendez vous ensuite sur la plateforme de centre expert
:https://antflow-iac.geo-platformlab.fr/ui/


Exemple animé
^^^^^^^^^^^^^^

.. thumbnail:: images/Plat_incub_launch_bench.gif

Etapes
^^^^^^^
Récupérer l'url de votre job lancé précédemment:

.. thumbnail:: images/tutorial/launched_job.png

Cliquez sur "Ouvrir OGC API" et copier le lien correspondant:

.. thumbnail:: images/tutorial/open_ogc_api_button.png

.. thumbnail:: images/tutorial/api_ogc_link.png

Cliquer sur le projet **auto_bench_cwl**

.. thumbnail:: images/tutorial/auto_cwl_proj_antflow.png

Lancer le traitement **bench**

.. thumbnail:: images/tutorial/bench_cwl.png

Remplir les informations suivantes:

.. thumbnail:: images/tutorial/bench_input.png

* **antflow_link**: Url du job récupérée précédemment
* **bucket_name** (facultatif): Nom du bucket contenant le fichier de configuration vu plus haut
* **git_s3_map_s3_path**: Nom du fichier de configuration dans le bucket, chemin relatif à la racine du bucket,
  il n'est pas nécéssaire de re-inscrire le nom du bucket dans le chemin. (ex: *monfichier.json*, si à la racine)

Exemple complet:

.. thumbnail:: images/tutorial/bench_input_filled.png

Accès au résultats
~~~~~~~~~~~~~~~~~~

Une fois le job de benchmark effectué il est possible d'accéder au
résultats de la manière suivante:

.. thumbnail:: images/tutorial/bench_done.png

.. thumbnail:: images/tutorial/bench_output.png
