caktus.django-k8s
=================

An Ansible role with sane defaults to deploy a Django app to Kubernetes.

It also tries to arrange to be able to update the deploy from an automated
CI service without requiring manual intervention for authentication.

License
~~~~~~~~~~~~~~~~~~~~~~

This Ansible role is released under the BSD License.  See the `LICENSE
<https://github.com/caktus/ansible-role-aws-web-stacks/blob/master/LICENSE>`_
file for more details.

Development sponsored by `Caktus Consulting Group, LLC
<http://www.caktusgroup.com/services>`_.


Requirements
~~~~~~~~~~~~~~~~~~~~~~

* ``pip install openshift kubernetes-validate``


Installation
------------

1. Add to your ``requirements.yml``:

.. code-block:: yaml

    ---
    # file: deployment/requirements.yaml

    - src: https://github.com/caktus/ansible-role-django-k8s
      name: caktus.django-k8s

2. Add the role to your playbook:

.. code-block:: yaml

    ---
    # file: deploy.yaml

    - hosts: k8s_clusters
      vars:
        ansible_python_interpreter: "{{ ansible_playbook_python }}"
      roles:
      - role: caktus.django-k8s

3. Create an inventory file for your clusters:

.. code-block:: yaml

    ---
    # file: inventory.yaml

    all:
      children:
        k8s_clusters:
          vars:
            ansible_connection: local
          hosts:
            gcp-staging:
              k8s_kube_context: <name of context from ~/.kube/config>
              k8s_domain_names:
              - www.example.com

See ``defaults/main.yml`` for all the variables that can be overridden.

Usage
-----

This should be run first interactively by a user who is already set up to access the
cluster using kubectl. E.g., they can run `kubectl cluster-info` and see the
cluster info. How to achieve that will differ by Kubernetes hosting environment.

When run the first time, this will figure out some information using the
user's kubectl access that the user will need to save in Ansible variables
for later use by the CI test service.

This role will also create a "deploy account" in Kubernetes which has the
necessary permissions to deploy. It will print out a long random
string which is the secret that can be used later to do Kubernetes activities
using that deploy account (e.g. during CI tests and deploys).

Follow the instructions that are printed during that first run (putting some
information into variables and files). Then run again, and this time it should
complete successfully having created the various K8S objects.

Configuration
-------------


Celery
``````

.. code-block:: yaml

  # Required to enable:
  k8s_worker_enabled: true
  k8s_worker_celery_app: "<app.celery.name>"
  k8s_worker_beat_enabled: true  # only if beat is needed

  # Optional variables (with defaults):
  k8s_worker_replicas: 2
  k8s_worker_image: "{{ k8s_container_image }}"
  k8s_worker_image_pull_policy: "{{ k8s_container_image_pull_policy }}"
  k8s_worker_image_tag: "{{ k8s_container_image_tag }}"
  k8s_worker_resources: "{{ k8s_container_resources }}"
