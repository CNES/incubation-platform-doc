Erreur connues
=============================

Nom de l'image de conteneur incorrecte, permission d'accéder à la registry refusée
------------------------------------------------------------------------------------

Dans le cas ou le cluster n'arrive pas à télécharger l'image conteneur de vos étapes CWL vous aurez l'erreur suivante dans les logs au sein d'Antflow.

.. code:: bash

    ERROR Pod stuck waiting, reason: ErrImagePull, pod: run-get-slope-file-pod-yrmedjwq, check that your container image is correct (name, permission)
    ERROR Pod stuck waiting, reason: ErrImagePull, pod: run-get-conf-file-pod-tawdpkgz, check that your container image is correct (name, permission)
    ERROR Pod stuck waiting, reason: ErrImagePull, pod: run-get-pekel-file-pod-crohlpok, check that your container image is correct (name, permission)
    ERROR Pod stuck waiting, reason: ErrImagePull, pod: run-get-l2a-pod-kfuujkyt, check that your container image is correct (name, permission)
    ERROR Pod stuck waiting, reason: ErrImagePull, pod: run-get-water-config-file-pod-natetbgx, check that your container image is correct (name, permission)
    ERROR Pod stuck waiting, reason: ErrImagePull, pod: run-get-hand-file-pod-oixsodvq, check that your container image is correct (name, permission)

Veuillez vérifier que vous avez respecté les préconisations données ici: :ref:`hints_requirement`