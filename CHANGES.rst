caktus.django-k8s
=================


Changes
-------


v0.0.2 on Apr 15, 2020
~~~~~~~~~~~~~~~~~~~~~~

* Made some changes to simplify setting up a deploy account so this can be run from
  continuous integration.

  *If updating from v0.0.1*:

  * ``k8s_auth_host`` is now a required variable - see the README.rst.
  * After setting that, please run first locally with kubectl set up
    to access the cluster, and follow any instructions that are output.


v0.0.1 on Mar 26, 2020
~~~~~~~~~~~~~~~~~~~~~~

* Initial release
