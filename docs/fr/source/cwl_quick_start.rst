CWL Quick Start
===============

Ceci est un guide rapide sur le lancement d'un chaîne de traitement à
l'aide d'un workflow CWL.

1.1. Chaîne “auto portante”
---------------------------

Prenons la chaine suivante:

.. thumbnail:: images/cwl_quick_start/simple_chain.drawio.png

Pas de fichiers en entré, uniquement des arguments

Tout CWL commence par la définition suivante:

.. code:: yaml

  cwlVersion: v1.2
  $namespaces:
    s: https://schema.org/
    js: https://json-schema.org/
  $schemas:
  - http://schema.org/version/latest/schemaorg-current-http.rdf

1.1.1. Entrées du workflow
~~~~~~~~~~~~~~~~~~~~~~~~~~

Nous allons ensuite définir notre workflow, on commence par quelques
informations et les entrées, ici nous en avons deux, ce sont des
strings:

.. code:: yaml

  $graph:
  - class: Workflow
    id: main
    label: Mon workflow
    doc: Un workflow avec une seule étape
    inputs:
      stac_items:
          type: string
          doc: Lien vers l'item STAC à utiliser
      mode:
          type: string
          doc: Le mode de la chaîne

Il possible de choisir **plusieurs types d'entrée**, par exemple:

.. code:: yaml

    inputs:
      mode:
          type: ["string", "boolean"]
          doc: Lien vers l'item STAC à utiliser 

Mais aussi une valeur **par défaut:**

.. code:: yaml

    inputs:
      mode:
          type: [string, boolean]
          doc: Lien vers l'item STAC à utiliser
          default: fast

Ou une valeur **optionnelle**:

.. code:: yaml

    inputs:
      mode:
          type: ["null", string, boolean]
          doc: Lien vers l'item STAC à utiliser

1.1.2. Étapes
~~~~~~~~~~~~~

On spécifie maintenant les étapes de notre workflow, nous en avons une
seule:

.. code:: yaml

    steps:
      run_chain:
        run: '#run_chain_step'
        in:
          stac_items: stac_items
          mode: mode
        out:
            [products, metadata]

**Chaque étapes possède ses propres entrées**, ici nous avons un cas
facile, les entrées du workflow vues plus haut sont directement injectés
dans les entrées de l'étape.

Le nom de la /des sorties devra/devrons correspondre à celui/ceux
présent dans la spécification de l'étape (voir plus bas).

1.1.3. Sorties du workflow
~~~~~~~~~~~~~~~~~~~~~~~~~~

Nous spécifions les sorties du workflow, ici aussi un cas simple, nous
avons qu'une seule étape donc les sorties du workflow correspond aux
sorties de cette dernière.

.. code:: yaml

    outputs:
      product_out:
          type: Directory
          outputSource: run_chain/products
      metadata_out:
          type: File
          outputSource: run_chain/metadata

Attention à bien spécifier le type de sortie correspondant au type de la
sortie de l'étape (ici run_chain). Voir plus bas pour les specs de la
sortie de l'étape.

1.1.4. Spécification de l'étape
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Voici comment spécifier notre étape:

.. code:: yaml

 - class: CommandLineTool
   id: run_chain
   baseCommand: "run"
   arguments: ["--out", $(runtime.outdir)]
   hints:
     DockerRequirement:
       dockerPull: registry.co/project/myimage:v1.2
     ResourceRequirement:
       ramMax: 3000
   requirements:
     NetworkAccess:
       networkAccess: true
   inputs:
     stac_items:
       type: string
       inputBinding:
         prefix: "--stac_link"
         position: 1
     mode:
       type: string
       inputBinding:
         prefix: "--mode"
         position: 2
   outputs:
     product_out:
       type: Directory
       outputBinding:
         glob: $(runtime.outdir)/*.tif
      metadata_out:
       type: File
       outputBinding:
         glob: $(runtime.outdir)/meta.json

1.1.4.1. BaseCommand
^^^^^^^^^^^^^^^^^^^^

.. code:: yaml

  baseCommand: "run"

C'est la commande qui va être lancé à l'intérieur du conteneur, les
arguments serons ajoutés ensuite.

1.1.4.2. Arguments
^^^^^^^^^^^^^^^^^^

.. code:: yaml

  arguments: ["--out", $(runtime.outdir)] 

Ce sont les arguments ajoutés à chaques exécution.

.. caution::

  Il est nécessaire de spécifier $(runtime.outdir) comme
  dossier de sortie pour les produits de votre chaîne, lors du
  lancement du CWL c'est uniquement ce chemin qui sera persisté
  jusqu'au bout du workflow avant que les produits soient publiés dans
  un bucket S3.

  Il est évidemment possible de créer un sous dossier dans
  $(runtime.outdir)       

.. _hints:

1.1.4.3. Hints
^^^^^^^^^^^^^^

.. code:: yaml

    hints:
      DockerRequirement:
        dockerPull: registry.co/project/myimage:v1.2
      ResourceRequirement:
        coresMin: .. (nbr of core)
        coresMax: .. (nbr of core)
        ramMin: 100  (MB)
        ramMax: 3000 (MB)

.. caution::

  Merci de spécifier les ressources max utilisé par votre
  chaîne, les ressources min sont des **ressources réservés** dans tout
  les cas, merci d'ajuster ses valeurs de manière raisonnable.

  NB: Un sous attribution de ressources CPU entrainera une augmentation
  de temps d'exécution, une sous attribution de la RAM entrainera un
  arrêt du conteneur (OOM).

1.1.4.4. Requirements
^^^^^^^^^^^^^^^^^^^^^

.. code:: yaml

    requirements:
      NetworkAccess:
        networkAccess: true 

Si votre conteneur doit accéder à internet, n'oublier pas d'ajouter
cette section

1.1.4.5. Inputs
^^^^^^^^^^^^^^^

.. code:: yaml

    inputs:
      stac_items:
        type: string
        inputBinding:
          prefix: "--stac_link"
          position: 1
      mode:
        type: string
        inputBinding:
          prefix: "--mode"
          position: 2 

Nous spécifions ici les inputs de l'étape et le préfixe utilisé à la
génération de la commande pour le conteneur ainsi que l'ordre

1.1.4.6. Outputs
^^^^^^^^^^^^^^^^

.. code:: yaml

    outputs:
      product_out:
        type: Directory
        outputBinding:
          glob: $(runtime.outdir)/*.tif
      metadata_out:
        type: File
        outputBinding:
          glob: $(runtime.outdir)/meta.json

Nous spécifions ici les outputs ainsi que l'expression qui va permettre
de catché les fichiers/dossiers que nous voulons exporter de cette
étape.

1.2. Chaîne non “auto-portante”
-------------------------------

.. thumbnail:: images/cwl_quick_start/non_auto_chain_1.png

.. thumbnail:: images/cwl_quick_start/non_auto_chain_2.png

.. caution::

  Une chaîne de traitement avec fichiers de configuration ne
  suit pas le modèle du CWL qui est sensé définir au sein de ses
  entrées, l'ensemble des paramètre de la chaîne. (OGC Package)

  Il semble donc préférable d'avoir uniquement des arguments pour ce
  qui est des paramètres de lancement de la chaîne.

  De plus, le lancement de CWL avec un input type “File” ou “Directory”
  au sein de la plateforme d'incubation est impossible.

+----------------------+-----------------+---------------------------------------+
| Fonction             | Type d'entrée   | Cible                                 |
|                      |                 |                                       |
|                      |                 |                                       |
+======================+=================+=======================================+
| Configuration du     | Fichier         | Arguments / argument templaté dans un |
| lancement de la      |                 | fichier temporaire au sein du CWL     |
| chaîne               |                 | (voir 1.2.1)                          |
+----------------------+-----------------+---------------------------------------+
| Produit d'entrée     | Fichier/Dossier | Dans bucket S3, accès direct depuis   |
|                      |                 | la chaîne ou étape préliminaire de    |
|                      |                 | téléchargement                        |
+----------------------+-----------------+---------------------------------------+
| Configuration        | Fichier         | Fichier dans bucket S3, accès direct  |
| utilisé par algo     |                 | depuis chaîne                         |
| durant exécution     |                 |                                       |
+----------------------+-----------------+---------------------------------------+

1.2.1. Fichier de config vers arguments sans modifier le code de la chaîne
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Il est possible en CWL de templater un fichier à partir d'arguments, ce
dernier est temporaire et peut être passé à votre chaîne, cela permet
d'assurer la compatibilité CWL sans modifier le code de la chaîne.

.. code:: yaml

    requirements:
      InlineJavascriptRequirement: {}
      InitialWorkDirRequirement:
        listing:
          - entryname: conf_file.json
            entry: |-
              { 
                "process_dirs" :
                  {
                  "input_dir": "$(inputs.input_dir.path)",
                  "output_dir": "$(runtime.outdir)/out_surf",
                  "tmp_dir": "$(runtime.tmpdir)"
                  },
                "auxiliary_data" :
                  {
                  "hand_file": "$(inputs.hand_file.path)",
                  "slope_file": "$(inputs.slope_file.path)",
                  "non_floodable_mask_path": $(inputs.non_floodable_mask_path ? inputs.non_floodable_mask_path.path : null),
                  "pekel_file": "$(inputs.pekel_file.path)",
                  "global_goas_file": $(inputs.global_goas_file ? inputs.global_goas_file.path : null),
                  "goas_mask_path": $(inputs.goas_mask_path ? inputs.goas_mask_path.path : null),
                  "waterdetect_config_file": "$(inputs.waterdetect_config_file ? inputs.waterdetect_config_file.path : null)"
                  },
                  "product" :
                  {
                  "date": "$(inputs.product_date)",
                  "tile": "$(inputs.product_tile)",        
                  "version": "$(inputs.product_version)",
                  "counter": "$(inputs.product_counter)"
                  },
                  "verbose": "$(inputs.log_level)"
                }

Vous avez accès aux inputs avec la clé “inputs”. Le fichier est par la
suite utilisable avec “arguments”:

.. code:: yaml

  arguments: ["surf_water", "--cmd", "cmd_file.json"]

1.2.2. Téléchargement préliminaire fichier en entrée
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Si la chaîne n'est pas capable d'accéder elle même aux produits
d'entrée, voici un exemple d'étape préliminaire CWL pour télécharger des
produits depuis un bucket s3

.. code:: yaml

 - class: CommandLineTool
   id: retrieve_slope_file
   baseCommand: download_from_s3
   arguments: ["--out", $(runtime.outdir)/slope]
   hints:
     DockerRequirement:
       dockerPull: artifactory.cnes.fr/platform-incub-docker-prod-local/cwl-processing/cwl_helpers:v1.3.3
     ResourceRequirement:
       ramMin: 200
   requirements:
     NetworkAccess:
       networkAccess: true
   inputs:
     bucket_name:
       type: string
       inputBinding:
         prefix: "--bucket"
         position: 1
     file_path:
       type: string
       inputBinding:
         prefix: "--path"
         position: 2
   outputs:
     slope_out:
       type: File/Directory
       outputBinding:
         glob: $(runtime.outdir)/slope/*.tif
         OU
         glob: $(runtime.outdir)/slope/

.. caution::
  Le chemin est relatif au bucket, pas besoin de re écrire
  son nom.

  Si vous télécharger un dossier, ne pas oublier de mettre un “/” à la
  fin du chemin

  Voir info
  ici: https://gitlab.cnes.fr/plateforme-incubation/cwl-processing/cwl_helpers


Pour utiliser: *artifactory.cnes.fr/platform-incub-docker-prod-local/cwl-processing/cwl_helpers:v1.3.3*
ajouter les variables CI/CD suivantes au projet qui contient votre CWL **en mode visible**:

.. thumbnail:: images/cwl_quick_start/helper_ci_variables.png

1.2.3. Téléchargement préliminaire des produits d'entrée avec EODAG
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: yaml

 - class: CommandLineTool
   id: retrive_l2a_data
   baseCommand: eodag
   arguments: ["download", "--output-dir", $(runtime.outdir)]
   hints:
     DockerRequirement:
       dockerPull: artifactory.cnes.fr/platform-incub-docker-prod-local/cwl-processing/cwl_helpers:v1.3.3
     ResourceRequirement:
       ramMin: 200
   requirements:
     NetworkAccess:
       networkAccess: true
   inputs:
     stac_link:
       type: string
       inputBinding:
         prefix: "--stac-item"
         position: 1
   outputs:
     l2a_out:
       outputBinding:
         glob: $(runtime.outdir)/*
       type: Directory

Les arguments correspondent à la commande EODAG, plus d'info ici:
https://eodag.readthedocs.io/en/stable/cli_user_guide.html

Voici à quoi ressemblerais les variables CI/CD avec le provider GEODES:

.. thumbnail:: images/cwl_quick_start/helper_ci_variables.png
