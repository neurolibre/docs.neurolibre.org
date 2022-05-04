Bare-metal to BinderHub
=======================

Installation of the BinderHub  from bare-metal is fully automatic and reproducible through `terraform <https://www.terraform.io/>`_ configuration
runned using `this Docker container <https://github.com/neurolibre/neurolibre-binderhub/blob/master/Dockerfile>`_.

The following is intended for neurolibre backend developpers, but can be read by anyone interrested in our process.
It assumes that you have basic knowledge on using the command line on a remote server (bash, ssh authentification..).

The sections :ref:`Pre-setup` and :ref:`Docker-specific preparations` should be done just the first time.
Once it is done, you can directly go to the section :ref:`Spawn a BinderHub instance using Docker`.

Pre-setup
---------

You first need to prepare the necessary files that will be used later to install and ssh to the newly spawned BinderHub  instance.

We are using `git-crypt <https://github.com/AGWA/git-crypt>`_ to encrypt our password files for the whole process, these can be uncrypted with the appropriate :code:`gitcrypt-key`.
For the ssh authentication on the BinderHub  server, you have two choices : i) use neurolibre’s key (recommended) or ii) use your own ssh key.

.. note:: You can request the :code:`gitcrypt-key` and :code:`neurolibre’s ssh key` to any infrastructure admin if authorized.
.. warning:: You should never share the :code:`gitcrypt-key` and :code:`neurolibre’s ssh key` to anyone.

1. Create a folder on your local machine, which is later to be mounted to the Docker container for securely using your keys during spawning a BinderHub instance.
   Here, we will call it :code:`my-keys` for convenience:

   .. code-block:: console

     cd /home/$USER
     mkdir /my-keys

2. Option (i), use neurolibre’s key (recommended):

   a. Simply copy the public :code:`id_rsa.pub` and private key :code:`id_rsa` to :code:`/home/$USER/my-keys/`

      .. code-block:: console

        cp id_rsa* /home/$USER/my-keys/

3. Option (ii), use your own local key:

   a. Make sure your public key and private are under :code:`/home/$USER/.ssh` an copy it to :code:`/home/$USER/my-keys`.

      .. code-block:: console

        cp /home/$USER/.ssh/id_rsa* /home/$USER/my-keys/

   b. If not already associated, add your local's key to your GitHub account:

      * You can check and add new keys on your `GitHub settings <https://github.com/settings/keys>`_.
      * Test your ssh connection to your GitHub account by following `these steps <https://help.github.com/en/github/authenticating-to-github/testing-your-ssh-connection>`_.

4. Finally, copy the key :code:`gitcrypt-key` in :code:`/home/$USER/my-keys/`.

Docker-specific preparations
----------------------------

You will install a trusted Docker image that will later be used to spawn the BinderHub instance.

1. Install `Docker <https://www.Docker.com/get-started>`_ and log in to the dockerhub with your credentials.

   .. code-block:: console

     sudo docker login

2. Pull the Docker image that encapsulates the barebones environment to spawn a BinderHub instance with our provider (compute canada as of late 2019).
   You can check the different tags available under our `dockerhub user <https://hub.Docker.com/r/conpdev/neurolibre-instance/tags>`_.

   .. code-block:: console

     sudo docker pull conpdev/neurolibre-instance:v1.3

Spawn a BinderHub instance using Docker
---------------------------------------

To achieve this, you will instantiate a container (from the image you just pulled) mounted with specific volumes from your computer.
You will be mounting two directories into the container: :code:`/my_keys` containing the files from :ref:`Pre-setup`, and :code:`/instance_name` containing the terraform recipe and artifacts.

.. warning:: The Docker container that you will run contain sensitive information (i.e. your ssh keys, passwords, etc), so never share it with anyone else.
             If you need to share information to another developer, share the Dockerfile and/or these instructions.
.. note:: The Docker image itself has no knowledge of the sensitive files since they are used just at runtime
             (through `entrypoint <https://docs.docker.com/engine/reference/run/#entrypoint-default-command-to-execute-at-runtime>`_ command).

1. Place a :code:`main.tf` file (see :ref:`Appendix A` for details) into a new folder :code:`/instance-name`, which describes the terraform recipe for spawning a BinderHub instance on the cloud provider.
   For convenience, we suggest that you use the actual name of the instance (value of the :code:`project_name` field in :code:`main.tf`).

   .. code-block:: console

     mkdir /home/$USER/instance-name
     vim /home/$USER/instance-name/main.tf

.. note:: If you choose not to copy :code:`main.tf` file to this directory, you will be asked to fill out one manually during container runtime.

2. Start the Docker container which is going to spawn the BinderHub instance:

   .. code-block:: console

     sudo docker run -v /home/$USER/my_keys:/tmp/.ssh -v /home/$USER/instance-name:/terraform-artifacts -it neurolibre-instance:v1.2

3. Take a coffee and wait! The instance should be ready in 5~10 minutes.

4. For security measure, stop and delete the container that you used to span the instance:

   .. code-block:: console

     sudo docker stop conpdev/neurolibre-instance:v1.3
     sudo docker rm conpdev/neurolibre-instance:v1.3

If you need more information about this docker, check `the neurolibre repository <https://github.com/neurolibre/neurolibre-binderhub>`_.

Appendix A
----------

Here we describe the default terraform recipe that can be used to spawn a BinderHub  instance, it is also available `online <https://github.com/neurolibre/neurolibre-binderhub/blob/master/terraform/main.tf>`_.
There are three different modules used by our terraform scripts, all run consecutively and only if the previous one succeeded.

1. :code:`provider` populates terraform with the variables related to our cloud provider (compute canada as of late 2019):

    * :code:`project_name`: name of the instances (will be :code:`project_name_master` and :code:`project_name_nodei`)
    * :code:`nb_nodes`: number of k8s nodes **excluding** the master node
    * :code:`instance_volume_size`: main volume size of the instances in GB **including** the master node
    * :code:`ssh_authorized_keys`: list of the public ssh keys that will be allowed on the server
    * :code:`os_flavor_master`: hardware configuration of the k8s master instance in the form :code:`c{n_cpus}-{ram}gb-{optionnal_vol_in_gb}`
    * :code:`os_flavor_node`: hardware configuration of the k8s node instances
    * :code:`image_name`: OS image name used by the instance
    * :code:`docker_registry`: domain for the Docker registry, if empty it uses :code:`Docker.io` by default
    * :code:`docker_id`: user id credential to connect to the Docker registry
    * :code:`docker_password`: password credential to connect to the Docker registry

.. warning:: The flavors and image name are not fully customizable and should be set accordingly to the provider's list.
             You can check them through openstack API using :code:`openstack flavor list && openstack image list` or using the horizon dashboard.

2. :code:`dns` related to cloudflare DNS configuration:

    * :code:`domain`: domain name to access your BinderHub  environment, it will automatically point to the k8s master floating IP

3. :code:`binderhub` specific to binderhub configuration:

    * :code:`binder_version`: you can check the current BinderHub  version releases `here <https://jupyterhub.github.io/helm-chart/>`_
    * :code:`TLS_email`: this email will be used by `Let's Encrypt <https://letsencrypt.org/>`_ to request a TLS certificate
    * :code:`TLS_name`: TLS certificate name should be the same as the domain but with dashes :code:`-` instead of points :code:`.`
    * :code:`mem_alloc_gb`: Amount of RAM (in GB) used by each user of your BinderHub
    * :code:`cpu_alloc`: Number of CPU cores
      (`Intel® Xeon® Gold 6130 <https://ark.intel.com/content/www/us/en/ark/products/120492/intel-xeon-gold-6130-processor-22m-cache-2-10-ghz.html>`_
      for compute canada) used by each user of your BinderHub

.. code-block:: console
   :linenos:

    module "provider" {
    source = "git::ssh://git@github.com/neurolibre/terraform-binderhub.git//terraform-modules/providers/openstack"

    project_name         = "instance-name"
    nb_nodes             = 1
    instance_volume_size = 100
    ssh_authorized_keys  = ["<redacted>"]
    os_flavor_master     = "c4-30gb-83"
    os_flavor_node       = "c16-60gb-392"
    image_name           = "Ubuntu-18.04.3-Bionic-x64-2020-01"
    is_computecanada     = true
    docker_registry      = "binder-registry.conp.cloud"
    docker_id            = "<redacted>"
    docker_password      = "<redacted>"
    }

    module "dns" {
    source = "git::ssh://git@github.com/neurolibre/terraform-binderhub.git//terraform-modules/dns/cloudflare"

    domain    = "instance-name.conp.cloud"
    public_ip = "${module.provider.public_ip}"
    }

    module "binderhub" {
    source = "git::ssh://git@github.com/neurolibre/terraform-binderhub.git//terraform-modules/binderhub"

    ip               = "${module.provider.public_ip}"
    domain           = "${module.dns.domain}"
    admin_user       = "${module.provider.admin_user}"
    binder_version   = "v0.2.0-n121.h6d936d7"
    TLS_email        = "<redacted>"
    TLS_name         = "instance-name-conp-cloud"
    mem_alloc_gb     = 4
    cpu_alloc        = 1
    docker_registry  = "${module.provider.docker_registry}"
    docker_id        = "${module.provider.docker_id}"
    docker_password  = "${module.provider.docker_password}"
    }
