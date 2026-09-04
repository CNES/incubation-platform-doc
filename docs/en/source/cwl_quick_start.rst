CWL Quick Start
===============

This is a quick guide to launching a processing chain using a CWL workflow.

1.1. Self-contained Chain
-------------------------

Let's take the following chain as an example:

.. thumbnail:: images/cwl_quick_start/simple_chain.drawio.png

No input files, only arguments.

Every CWL starts with the following definition:

.. code:: yaml

  cwlVersion: v1.2
  $namespaces:
    s: https://schema.org/
    js: https://json-schema.org/
  $schemas:
  - http://schema.org/version/latest/schemaorg-current-http.rdf

1.1.1. Workflow Inputs
~~~~~~~~~~~~~~~~~~~~~~~~~~

We will now define our workflow, starting with some information and inputs. Here, we have two inputs, both strings:

.. code:: yaml

  $graph:
  - class: Workflow
    id: main
    label: My workflow
    doc: A workflow with a single step
    inputs:
      stac_items:
          type: string
          doc: Link to the STAC item to use
      mode:
          type: string
          doc: The mode of the chain

It is possible to choose **multiple input types**, for example:

.. code:: yaml

    inputs:
      mode:
          type: ["string", "boolean"]
          doc: Link to the STAC item to use

As well as a **default value**:

.. code:: yaml

    inputs:
      mode:
          type: [string, boolean]
          doc: Link to the STAC item to use
          default: fast

Or an **optional value**:

.. code:: yaml

    inputs:
      mode:
          type: ["null", string, boolean]
          doc: Link to the STAC item to use

1.1.2. Steps
~~~~~~~~~~~~~

We now specify the steps of our workflow; here, we have only one:

.. code:: yaml

    steps:
      run_chain:
        run: '#run_chain_step'
        in:
          stac_items: stac_items
          mode: mode
        out:
            [products, metadata]

**Each step has its own inputs**; in this case, it's straightforward, as the workflow inputs are directly injected into the step inputs.

The name(s) of the output(s) must match those specified in the step definition (see below).

1.1.3. Workflow Outputs
~~~~~~~~~~~~~~~~~~~~~~~~~~

We specify the workflow outputs; in this simple case, the workflow outputs correspond to the outputs of its single step:

.. code:: yaml

    outputs:
      product_out:
          type: Directory
          outputSource: run_chain/products
      metadata_out:
          type: File
          outputSource: run_chain/metadata

Be sure to specify the output type corresponding to the type of the step's output (here, `run_chain`). See below for the step output specifications.

1.1.4. Step Specification
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Here’s how to specify our step:

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

This is the command that will be executed inside the container; arguments will be added afterward.

1.1.4.2. Arguments
^^^^^^^^^^^^^^^^^^

.. code:: yaml

  arguments: ["--out", $(runtime.outdir)]

These are the arguments added to each execution.

.. caution::

  It is necessary to specify `$(runtime.outdir)` as the output directory for your chain's products. During CWL execution, only this path will be persisted until the end of the workflow before the products are published to an S3 bucket.

  It is, of course, possible to create a subdirectory within `$(runtime.outdir)`.

.. _hints:

1.1.4.3. Hints
^^^^^^^^^^^^^^

.. code:: yaml

    hints:
      DockerRequirement:
        dockerPull: registry.co/project/myimage:v1.2
      ResourceRequirement:
        coresMin: .. (number of cores)
        coresMax: .. (number of cores)
        ramMin: 100  (MB)
        ramMax: 3000 (MB)

.. caution::

  Please specify the maximum resources used by your chain. The minimum resources are **reserved** in all cases; adjust these values reasonably.

  Note: Underallocating CPU resources will increase execution time, while underallocating RAM will cause the container to stop (OOM).

1.1.4.4. Requirements
^^^^^^^^^^^^^^^^^^^^^

.. code:: yaml

    requirements:
      NetworkAccess:
        networkAccess: true

If your container needs internet access, do not forget to add this section.

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

Here, we specify the step inputs and the prefix used to generate the command for the container, as well as the order.

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

Here, we specify the outputs and the expression that will capture the files/directories we want to export from this step.

1.2. Non-Self-Contained Chain
-----------------------------

.. thumbnail:: images/cwl_quick_start/non_auto_chain_1.png

.. thumbnail:: images/cwl_quick_start/non_auto_chain_2.png

.. caution::

  A processing chain with configuration files does not follow the CWL model, which is supposed to define all chain parameters within its inputs (OGC Package).

  It is therefore preferable to have only arguments for chain launch parameters.

  Additionally, launching a CWL with an input type "File" or "Directory" within the incubation platform is impossible.

+----------------------+-----------------+---------------------------------------+
| Function             | Input Type      | Target                                |
|                      |                 |                                       |
+======================+=================+=======================================+
| Chain launch         | File            | Arguments / templated argument in a   |
| configuration        |                 | temporary file within CWL (see 1.2.1) |
+----------------------+-----------------+---------------------------------------+
| Input product        | File/Directory  | In S3 bucket, direct access from      |
|                      |                 | chain or preliminary download step    |
+----------------------+-----------------+---------------------------------------+
| Configuration used   | File            | File in S3 bucket, direct access      |
| by algorithm during  |                 | from chain                            |
| execution            |                 |                                       | 
+----------------------+-----------------+---------------------------------------+

1.2.1. Configuration File to Arguments Without Modifying Chain Code
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In CWL, it is possible to template a file from arguments. This file is temporary and can be passed to your chain, ensuring CWL compatibility without modifying the chain code.

.. code:: yaml

    requirements:
      InlineJavascriptRequirement: {}
      InitialWorkDirRequirement:
        listing:
          - entryname: conf_file.json
            entry: |-
              {
                "process_dirs":
                  {
                  "input_dir": "$(inputs.input_dir.path)",
                  "output_dir": "$(runtime.outdir)/out_surf",
                  "tmp_dir": "$(runtime.tmpdir)"
                  },
                "auxiliary_data":
                  {
                  "hand_file": "$(inputs.hand_file.path)",
                  "slope_file": "$(inputs.slope_file.path)",
                  "non_floodable_mask_path": $(inputs.non_floodable_mask_path ? inputs.non_floodable_mask_path.path : null),
                  "pekel_file": "$(inputs.pekel_file.path)",
                  "global_goas_file": $(inputs.global_goas_file ? inputs.global_goas_file.path : null),
                  "goas_mask_path": $(inputs.goas_mask_path ? inputs.goas_mask_path.path : null),
                  "waterdetect_config_file": "$(inputs.waterdetect_config_file ? inputs.waterdetect_config_file.path : null)"
                  },
                  "product":
                  {
                  "date": "$(inputs.product_date)",
                  "tile": "$(inputs.product_tile)",
                  "version": "$(inputs.product_version)",
                  "counter": "$(inputs.product_counter)"
                  },
                  "verbose": "$(inputs.log_level)"
                }

You can access inputs with the key "inputs." The file can then be used with "arguments":

.. code:: yaml

  arguments: ["surf_water", "--cmd", "cmd_file.json"]

1.2.2. Preliminary Download of Input Files
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If the chain cannot access input products itself, here is an example of a preliminary CWL step to download products from an S3 bucket:

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
         OR
         glob: $(runtime.outdir)/slope/

.. caution::
  The path is relative to the bucket; no need to rewrite its name.

  If downloading a directory, do not forget to add a "/" at the end of the path.

  See info here: https://gitlab.cnes.fr/plateforme-incubation/cwl-processing/cwl_helpers

To use: *artifactory.cnes.fr/platform-incub-docker-prod-local/cwl-processing/cwl_helpers:v1.3.3*
Add the following CI/CD variables to the project containing your CWL **in visible mode**:

.. thumbnail:: images/cwl_quick_start/helper_ci_variables.png

1.2.3. Preliminary Download of Input Products with EODAG
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

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

The arguments correspond to the EODAG command; more info here:
https://eodag.readthedocs.io/en/stable/cli_user_guide.html

Here’s what the CI/CD variables would look like with the GEODES provider:

.. thumbnail:: images/cwl_quick_start/helper_ci_variables.png
