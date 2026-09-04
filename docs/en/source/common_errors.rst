Known errors
=============================

Incorrect container image name or registry acces denied
------------------------------------------------------------------------------------

The below error will appear if the cluster couldn't pull the CWL step container image

.. code:: bash

    ERROR Pod stuck waiting, reason: ErrImagePull, pod: run-get-slope-file-pod-yrmedjwq, check that your container image is correct (name, permission)
    ERROR Pod stuck waiting, reason: ErrImagePull, pod: run-get-conf-file-pod-tawdpkgz, check that your container image is correct (name, permission)
    ERROR Pod stuck waiting, reason: ErrImagePull, pod: run-get-pekel-file-pod-crohlpok, check that your container image is correct (name, permission)
    ERROR Pod stuck waiting, reason: ErrImagePull, pod: run-get-l2a-pod-kfuujkyt, check that your container image is correct (name, permission)
    ERROR Pod stuck waiting, reason: ErrImagePull, pod: run-get-water-config-file-pod-natetbgx, check that your container image is correct (name, permission)
    ERROR Pod stuck waiting, reason: ErrImagePull, pod: run-get-hand-file-pod-oixsodvq, check that your container image is correct (name, permission)

Please check that you followed the steps specified here: :ref:`hints_requirement`