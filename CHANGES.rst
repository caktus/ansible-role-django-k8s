caktus.django-k8s
=================

Changes
-------

v1.10.0 on May 19, 2025
~~~~~~~~~~~~~~~~~~~~~~~

* Update default version memcached to 1.6.38, pgbouncer to 1.24.1, and redis to 7.4.3.

v1.9.0 on February 26, 2024
~~~~~~~~~~~~~~~~~~~~~

* Conditinally create or update registry secret on batchjobs.

v1.8.0 on October 24, 2023
~~~~~~~~~~~~~~~~~~~~~

* `k8s_pgbouncer_selfsigned_certificate_name`: add support mounting a
  Certificate from the `k8s_na````¡mespace` to pgBouncer's `/etc/pgbouncer/ssl/`
  directory to enable TLS mode to use for connections from clients
  (`client_tls_sslmode`).


v1.7.0 on September 27th, 2023
~~~~~~~~~~~~~~~~~~~~~

* This PR adds support for pgbouncer to help reduce Postgres connection usage and latency.


v1.6.0 on April 25th, 2023
~~~~~~~~~~~~~~~~~~~~~

* Manually create a non-expiring ``ServiceAccount`` token ``Secret`` to support Kubernetes v1.24. Relevant `release notes <https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.24.md#urgent-upgrade-notes>`_:

    * The ``LegacyServiceAccountTokenNoAutoGeneration`` feature gate is beta, and enabled by default. When enabled, Secret API objects containing service account tokens are no longer auto-generated for every ``ServiceAccount``. Use the `TokenRequest <https://kubernetes.io/docs/reference/kubernetes-api/authentication-resources/token-request-v1>`_ API to acquire service account tokens, or if a non-expiring token is required, create a Secret API object for the token controller to populate with a service account token by following this `guide <https://kubernetes.io/docs/concepts/configuration/secret/#service-account-token-secrets>`_. 

    * Each service account API key will be regenerated and will require you to rotate the ``k8s_auth_api_key`` Ansible variable(s).

* Switch to ``kubernetes.core`` for Ansible 6.x+ support and use `fully qualified collection names (FQCNs) <https://github.com/ansible-collections/overview/blob/4e7fdd2512a4ec213b1beccef3b58dfb58b0d06e/README.rst#terminology>`_ to be explicit. The ``community.kubernetes`` collection was renamed to ``kubernetes.core`` in `v2.0.0 of the kubernetes.core collection <https://github.com/ansible-collections/community.kubernetes/blob/main/CHANGELOG.rst#v2-0-0>`_. Since `Ansible v3.0.0 <https://github.com/ansible-community/ansible-build-data/blob/main/3/CHANGELOG-v3.rst#included-collections>`_, both the ``kubernetes.core`` and ``community.kubernetes`` namespaced collections were included for convenience. `Ansible v6.0.0 <https://github.com/ansible-community/ansible-build-data/blob/f3602822e899015312852bb3e2debe52df109135/6/CHANGELOG-v6.rst#L4281>`_ removed the ``community.kubernetes`` convenience package.

v1.5.2 on November 2nd, 2022
~~~~~~~~~~~~~~~~~~~~~
* Add support for customizing Deployment strategy for web and worker processes


v1.5.1 on May 24th, 2022
~~~~~~~~~~~~~~~~~~~~~
* Rename ``k8s_migration_command`` to ``k8s_migrations_command`` (the old name will continue
  working for now, but update your projects!)
* Add ``k8s_migrations_timeout`` variable
* Support further customizing batch jobs run before and after deploys via the new
  ``k8s_predeploy_batchjobs`` and ``k8s_postdeploy_batchjobs`` variables


v1.5.0 on April 20th, 2022
~~~~~~~~~~~~~~~~~~~~~

* Removed support for RabbitMQ. See `the README
  <https://github.com/caktus/ansible-role-django-k8s#rabbitmq-support>`_
  if you need to maintain an existing cluster.
* Support running multiple workers using ``k8s_worker_containers``
* Allow override of ``kubernetes.io/ingress.class`` using ``k8s_ingress_class``
* Allow ovverride Service ``spec.type`` and ``spec.loadBalancerIP`` for Memcache and Redis
* Add ``podAntiAffinity`` to prefer scheduling app and worker pods on different nodes
* Update Ingress ``apiVersion``  to ``networking.k8s.io/v1`` (Kubernetes 1.19+)
* Fix arg order for Celery 5.x
* Add support for mounting data volumes via Secrets within containers 


v1.4.0 on Oct 14, 2021
~~~~~~~~~~~~~~~~~~~~~~

* Add support for deploying a RabbitMQ cluster. See `the README
  <https://github.com/caktus/ansible-role-django-k8s#rabbitmq-support>`_
  for prerequisites.


v1.3.0 on Jun 18, 2021
~~~~~~~~~~~~~~~~~~~~~~

* Configure the `public access <https://docs.ansible.com/ansible/latest/collections/amazon/aws/s3_bucket_module.html#parameter-public_access>`_ block on private S3 bucket using `s3_bucket` module
  (requires Ansible 3.0+ or v1.3.0 of the amazon.aws collection)
* Add `skip_duplicates: false` to fix
  deprecation warnings and `set it to the default value
  <https://docs.ansible.com/ansible/latest/collections/community/aws/iam_policy_module.html#parameter-skip_duplicates>`_ for the following tasks:
    * aws_s3.yml's *Attach inline policy to user*
    * aws_ci.yml's *Attach inline policy to user*
* Support tagging S3 buckets


v1.2.0 on Mar 17, 2021
~~~~~~~~~~~~~~~~~~~~~~
* Bumps memcached version to 1.6.9


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
