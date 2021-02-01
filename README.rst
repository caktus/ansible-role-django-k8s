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

Web applications running on AWS typically use Amazon S3 for static and media
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
  # file: deploy-s3.yaml

  - hosts: k8s
    vars:
      ansible_connection: local
      ansible_python_interpreter: "{{ ansible_playbook_python }}"
    tasks:
      - name: configure Amazon S3 buckets
        import_role:
          name: caktus.django-k8s
          tasks_from: aws_s3

Run with: ``ansible-playbook deploy-s3.yaml``.


Amazon IAM: Adding a limited AWS IAM user for CI deploys
````````````````````````````````````````````````````````

In order to be able to deploy to AWS from CI systems, you'll need to be able to
authenticate as an IAM user that has the permissions to push to the AWS ECR (Docker
registry), and possibly need to be able to read a secret from AWS Secrets Manager (the
``.vault_pass`` value). This playbook can create that user for you with the proper
permissions. You can configure this with the following variables (defaults shown):

```yaml
k8s_ci_username: myproject-ci-user
k8s_ci_repository_arn: "" # format: arn:aws:ecr:<REGION>:<ACCOUNT_NUMBER>:repository/<REPO_NAME>
k8s_ci_vault_password_arn: "" # format: arn:aws:secretsmanager:<REGION>:<ACCOUNT_NUMBER>:secret:<NAME_OF_SECRET>
```

Only ``k8s_ci_repository_arn`` is required. The REPO_NAME portion can be found
[here](https://console.aws.amazon.com/ecr/repositories). The `k8s_ci_vault_password_arn`
is an optional pointer to a single secret in AWS Secrets Manager. The ARN can be found
by going to this [link](https://console.aws.amazon.com/secretsmanager/home#/listSecrets)
and then clicking on the secret you're sharing with the user. On some projects, we store
the Ansible vault password in SecretsManager and then use an AWS CLI command to read the
secret so other secrets in the repo can be decrypted. This allows the CI user to access
that command.

You'll need to create a separate playbook to invoke this functionality because, once
created, we don't need to try to recreate the user on each deploy AND because the CI
user will not have the permissions to create itself, so we don't want this playbook to
run on CI deploys. Create a playbook that looks like this:

.. code-block:: yaml

  ---
  # file: deploy-ci.yaml

  - hosts: k8s
    vars:
      ansible_connection: local
      ansible_python_interpreter: "{{ ansible_playbook_python }}"
    tasks:
      - name: configure CI IAM user
        import_role:
          name: caktus.django-k8s
          tasks_from: aws_ci

Normally we would just run this with ``ansible-playbook deploy-ci.yaml``, but
unfortunately the Ansible IAM role still uses boto (instead of boto3) and boto is not
compatible with using AWS profiles or AssumeRoles which we usually use to get access to
AWS subaccounts. So first, you'll have to run this python script, which takes your
profile (``saguaro-cluster`` in this example) and converts that into credentials that
boto can use. Here is the python script:

.. code-block:: python

   import boto3

   session = boto3.Session(profile_name="saguaro-cluster")
   credentials = session.get_credentials().get_frozen_credentials()

   print(f'export AWS_ACCESS_KEY_ID="{credentials.access_key}"')
   print(f'export AWS_SECRET_ACCESS_KEY="{credentials.secret_key}"')
   print(f'export AWS_SECURITY_TOKEN="{credentials.token}"')
   print(f'export AWS_SESSION_TOKEN="{credentials.token}"')


The script will print statements to your console. Copy and paste those into your console
and then run ``ansible-playbook deploy-ci.yaml`` and it should work.

After you run this role, the IAM user will be created with the proper permissions.
You'll then need to use the AWS console to create an access key and secret key for that
user. Take note of the `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` values.

Copy those 2 variables (and `AWS_DEFAULT_REGION`) into the CI environment variables
console.

NOTE: Be aware that you'll need to make sure that `k8s_rollout_after_deploy` is disabled
(which is the default), because the rollout commands use your local ``kubectl`` which
likely has more permissions than the IAM service account that this role depends on. See
https://github.com/caktus/ansible-role-django-k8s/issues/25.
