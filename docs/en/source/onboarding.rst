Onboarding - Incubation Platform
=============================

Available Resources
--------------------

The following resources are provided for the incubation center dedicated to workflow execution (**shared**):

* 4 DEV1-XL Nodes (4 CPU, 12GB RAM)
* 220Gi storage for temporary files during execution

Available resources may be subject to change.

.. warning::
   It is important to properly specify in your OGC package (CWL) the resources required to run your workflow, particularly CPU and memory limits.
   For more information, see: :ref:`hints`

.. _hints_requirement:

Prerequisites
-------------

* A CWL workflow with one or more tasks using containers and not depending on any POSIX shared storage.

   * Some useful components: :doc:`cwl_quick_start`

* Container images must be accessible from the incubation platform cluster. Three options are available:

   * Add the **platform-incub-token** group with read-only (RO) access to your Artifactory registry.
   * Push your image to: **platform-incub-docker-prod-local/cwl-processing** (rights added upon request).
   * Make your image **publicly accessible**.

* Have input data for running the workflow (upload to dev bucket, EODAG, etc.).
   * The S3 bucket can be on any provider
* A CNES GitLab account.
* Compliance with the following guideline: https://processing-chain-guidelines.readthedocs.io/en/latest/

Workflow
--------

Below is a macro diagram of the expert center usage steps.

Key points are as follows:

* Your development bucket is only used for input data for your workflow and by the benchmarking code.

   * The procedure for importing a secret passed as an environment variable in your workflow will be explained later.

* Output data from your workflow is stored in the Antflow-dedicated bucket. You can explore and download it using the STAC browser.
* Your CWL document can be stored in a dedicated repository or in the repository containing your code.

.. thumbnail:: images/centre_expert_workflow.png

Creating a Dedicated S3 Bucket for Development (Only with access to CNES network)
-----------------------------------------------

This bucket will allow you to store the following files:

* Any input file for your workflow, retrieved during execution either through a dedicated step (see: :doc:`CWL Quick Start`) or by the workflow itself.
* The benchmark configuration file (see below).

.. warning::
   You need to be able to access the CNES network to access the self-service platform

Go to the cloud self-service platform:
https://squest.admin.cloud.cnes.fr

Log in with your IPA account.

Create a service account with the "Service Account (Scaleway)" offer, you will get the access key on the web interface and the secret key by email. 

Select the "S3 personnalisé (Scaleway)" offer and create a bucket with your chosen name.

Use the id of the service account previously added to set permission to the bucket. 

For example: **application_id:<id> | s3:* | <bucket_name>, <bucket_name>/*** for full access (RW)

.. warning::
   Enter the following line in "custom_policy_lines" text box: **application_id:ef62a65b-b3c9-4e3f-8a1f-a6cf6cd23cf2 | s3:ListBucket, s3:GetObject | <bucket_name>, <bucket_name>/***

   This will allow the benchmark script to have read-only access to your bucket and retrieve the configuration file (see below).
   Follow the guide below for more details.

.. thumbnail:: images/s3_squest.gif

Launching Your CWL Workflow
---------------------------

For GitLab project configuration:
https://antflow-iac.geo-platformlab.fr/ui/#/docs/references/project-setup

.. warning::
   A few minutes (2-3) may be required before your process starts. This is due to the on-the-fly creation of compute nodes handling the execution of your workflow.

Adding Environment Variables
~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you wish to add environment variables during execution:
https://antflow-iac.geo-platformlab.fr/ui/#/docs/references/env-variables

.. warning::
   All variables must be prefixed with: "CWL__"
   Variables must be set to "visible" and "protected".

.. thumbnail:: images/CWL_project_pipeline_var.png

Launching the Compliance Benchmark
----------------------------------

.. warning::
   You have **3 days** to launch the benchmark for an executed workflow. After this period, the metrics will be deleted, and you will need to run your workflow again.

Configuration
~~~~~~~~~~~~~

Only one configuration file is required for the benchmark. Here is an example:

.. code:: python

   {
       "registry.gitlab.com/my_registry/project/image:tag": {
           "link": "https://gitlab.orfeo-toolbox.org/project/blaba",
           "token": ""  <!-- public project -->
       },
       "artifactory.cnes.fr/my_registry/project/image:tag": {
           "link": "https://gitlab.cnes.fr/Theia/surf-water",
           "token": "xxx"  <!-- private project -->
       }
   }

To obtain profiling metrics, the benchmark tool must be able to access the code executed in your workflow's container. For this, you need to specify the **container image and the corresponding repository**.

.. warning::
   The image name must match the one in your CWL workflow.

Read-only access with a GitLab Access Token is sufficient if your repository is protected.

**If you do not want profiling**, do not fill in the bucket name/path to the configuration file in the benchmark form.

Launching
~~~~~~~~~

Next, go to the expert center platform:
https://antflow-iac.geo-platformlab.fr/ui/

Animated Example
^^^^^^^^^^^^^^^^

.. thumbnail:: images/Plat_incub_launch_bench.gif

Steps
^^^^^
Retrieve the URL of your previously launched job:

.. thumbnail:: images/tutorial/launched_job.png

Click "Open OGC API" and copy the corresponding link:

.. thumbnail:: images/tutorial/open_ogc_api_button.png

.. thumbnail:: images/tutorial/api_ogc_link.png

Click on the **auto_bench_cwl** project:

.. thumbnail:: images/tutorial/auto_cwl_proj_antflow.png

Launch the **bench** process:

.. thumbnail:: images/tutorial/bench_cwl.png

Fill in the following information:

.. thumbnail:: images/tutorial/bench_input.png

* **antflow_link**: URL of the previously retrieved job
* **bucket_name** (optional): Name of the bucket containing the configuration file mentioned above
* **git_s3_map_s3_path**: Name of the configuration file in the bucket, relative path from the root of the bucket. There is no need to re-enter the bucket name in the path (e.g., *myfile.json* if at the root).

Complete Example:

.. thumbnail:: images/tutorial/bench_input_filled.png

Accessing Results
~~~~~~~~~~~~~~~~~

Once the benchmark job is completed, you can access the results as follows:

.. thumbnail:: images/tutorial/bench_done.png

.. thumbnail:: images/tutorial/bench_output.png
