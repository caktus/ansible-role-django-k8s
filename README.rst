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
              k8s_auth_host: <https://....Cluster API endpoint URL.....>
              k8s_domain_names:
              - www.example.com

See ``defaults/main.yml`` for all the variables that can be overridden.

The ``k8s_auth_host`` variable is absolutely required to be set. This is the API
endpoint URL of the cluster to use. Here are some examples so you can see what
it might look like:

AKS: ``https://ratom-staging-dns-ba5d6fd2.hcp.eastus.azmk8s.io:443``

AWS: ``https://74406E3AD450E7845D0EF653E7C6F020.gr7.us-west-2.eks.amazonaws.com``

Digital Ocean: ``https://fc22cd06-0dc4-4e19-a1cd-e0064d2d151e.k8s.ondigitalocean.com``

GKE: ``https://104.196.6.244``

Minikube: ``https://192.168.99.100:8443``

If you're sure you have ``kubectl`` set up to talk to your cluster, then you can run this to
print your ``k8s_auth_host`` value::

    kubectl config view --minify=true -o jsonpath='{.clusters[0].cluster.server}' --raw

Alternatively, if you used aws-web-stacks to create an EKS cluster, then the ``ClusterEndpoint``
output in CloudFormation is the value to use.

Usage
-----

This should be run first interactively by a user who is already set up to access the
cluster using kubectl. E.g., they can run ``kubectl cluster-info`` and see the
cluster info. How to achieve that will differ by Kubernetes hosting environment.

(If when you try to run this role the first time, you get a bunch of SSL errors,
check that ``k8s_auth_host`` and your current kubectl context are both pointing
to the same cluster.)

When run the first time, this will figure out some information using the
user's kubectl access that the user will need to save in Ansible variables
for later use by the CI test service.

This role will also create a "deploy account" in Kubernetes which has the
necessary permissions to deploy.

Follow the instructions that are printed during that first run (putting some
information into variables and files). Then run again, and this time it should
complete successfully having created the various K8S objects.

After that, the role should work without having to have kubectl access to the
cluster. The user or service running it just needs access to the Ansible vault
password, so ansible can decrypt the ``k8s_auth_api_key`` value.

Configuration
-------------

Review all of the variables in ``defaults/main.yml`` to see which configuration options
are available.

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


Amazon S3: IAM role for service accounts
````````````````````````````````````````

Django applications running on AWS typically use Amazon S3 for static and media
resources. ``caktus.django-k8s`` optionally supports enabling a Kubernetes
service account and associated IAM role that defines the access to public and
private S3 buckets. This provides similar functionality of
`EC2 instance profiles <https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2.html>`_
within Kubernetes namespaces. This
`AWS blog post <https://aws.amazon.com/blogs/opensource/introducing-fine-grained-iam-roles-service-accounts/>`_
also provides a good overview.

At a high level, the process is:

1. Create public and private S3 buckets
2. `Enable IAM roles for cluster service accounts <https://docs.aws.amazon.com/eks/latest/userguide/enable-iam-roles-for-service-accounts.html>`_
    * Requirement: `eksctl <https://eksctl.io/introduction/#installation>`_ must be installed
3. `Create an IAM role with a trust relatinoship and S3 policy for a service account <https://docs.aws.amazon.com/eks/latest/userguide/create-service-account-iam-policy-and-role.html>`_
4. `Annotate the service account with the ARN of the IAM role <https://docs.aws.amazon.com/eks/latest/userguide/specify-service-account-role.html>`_

Required variables:

  * ``k8s_s3_cluster_name``: name of EKS cluster in AWS

A separate playbook can be used to invoke this functionality:

.. code-block:: yaml

  ---
  # file: s3.yaml

  - hosts: k8s
    vars:
      ansible_connection: local
      ansible_python_interpreter: "{{ ansible_playbook_python }}"
    tasks:
      - name: configure Amazon S3 buckets
        import_role:
          name: caktus.django-k8s
          tasks_from: aws_s3
