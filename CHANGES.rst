caktus.django-k8s
=================

Changes
-------

v1.2.0 on TBD
~~~~~~~~~~~~~~~~~~~~~~

* Configure the [public access](https://docs.ansible.com/ansible/latest/collections/amazon/aws/s3_bucket_module.html#parameter-public_access) block on private S3 bucket using `s3_bucket` module
  (requires Ansible 3.0+ or v1.3.0 of the amazon.aws collection)
* Add `skip_duplicates: false` to *Attach inline policy to user* task to fix
  deprecation warning and set it to the default value.


v1.1.0 on Mar 4, 2021
~~~~~~~~~~~~~~~~~~~~~~
* Support adding a limited AWS IAM user for CI deploys


v1.0.0 on Feb 17, 2021
~~~~~~~~~~~~~~~~~~~~~~

**BACKWARDS INCOMPATIBLE CHANGES:**

* Use updated `cert-manager` annotation key: `cert-manager.io/cluster-issuer`
* Must update to [caktus.k8s-web-cluster](https://github.com/caktus/ansible-role-k8s-web-cluster) v1.0.0


v0.0.11 on Feb 2, 2021
~~~~~~~~~~~~~~~~~~~~~~
* Adds ``no_log`` to rollout commands to prevent logging of environment vars.


v0.0.10 on Jan 27, 2021
~~~~~~~~~~~~~~~~~~~~~~~
* Fixes migration bug (#35)
* Fixes deploy account lookup bug (#36)


v0.0.9 on Jan 4, 2021
~~~~~~~~~~~~~~~~~~~~~
* Fixes elasticsearch bug that did not allow pods to return to running state after deletion.


v0.0.8 on Sep 24, 2020
~~~~~~~~~~~~~~~~~~~~~~
* Add customizable ``k8s_collectstatic_timeout`` variable
* Suport redirect from www.domain.com to domain.com or vice versa.


v0.0.7 on Jul 28, 2020
~~~~~~~~~~~~~~~~~~~~~~
* Support environment-specific Amazon S3 bucket creation (#27)


v0.0.6 on Jul 2, 2020
~~~~~~~~~~~~~~~~~~~~~
* Allow full customization of the arguments to the celery command. (#17, #23)
* Enable ``collectstatic`` command to run during deploy (#24)


v0.0.5 on Jun 16, 2020
~~~~~~~~~~~~~~~~~~~~~~
* Add ``fsGroup`` to the beat service which allows that service to access the data
  volume, if it is not running as root.


v0.0.4 on Jun 15, 2020
~~~~~~~~~~~~~~~~~~~~~~
* Wait until Job-created migration pod returns ``Completed`` status before continuing
  deploy
* Set celery-beat ImagePullPolicy to match user-configured setting


v0.0.3 on Apr 28, 2020
~~~~~~~~~~~~~~~~~~~~~~
* If ``k8s_rollout_after_deploy`` is ``true``, use rollout to ensure that pods are restarted
  when we deploy. This ensures that even if our image tag is unchanged (like if
  we're using a branch name), we'll still pull the latest image with that tag and
  be running it when the deploy completes.


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
