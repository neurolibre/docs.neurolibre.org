Infrastructure details
======================

At the bottom of our infrastructure, we rely on `openstack <https://www.openstack.org/>`_ which spawns our multiple VMs (what we will reffer later as instance) and virtual volumes.
After successfull spawning of the instance, it is assigned a floating IP used to connect to it from the outside world.
The `cloudflare DNS <https://www.cloudflare.com/>`_ then properly configure the choosen domain name under :code:`*.conp.cloud` automatically pointing to the assigned floating IP.
When the network has been properly setup, the installation can continue with `kubernetes <https://kubernetes.io/>`_ and finishes with `BinderHub  <https://BinderHub .readthedocs.io/en/latest/index.html>`_.

We want to share our experience with the community, hence all our installation scripts are open-source available under
`neurolibre/kubeadm-boostrap <https://github.com/neurolibre/kubeadm-bootstrap>`_ and `neurolibre/terraform-binderhub <https://github.com/neurolibre/terraform-binderhub>`_.

.. warning:: NeuroLibre is still at an alpha stage of development, the github repositories will change frequently so be carefull if you use them.

Bare-metal to BinderHub
-----------------------

Installation of the BinderHub  from bare-metal is fully automatic and reproducible through `terraform <https://www.terraform.io/>`_ configuration
runned via `this <https://github.com/neurolibre/neurolibre-binderhub/blob/master/Dockerfile>`_ Docker container.

The following is intended for neurolibre backend developpers, but can be read by anyone interrested in our process.
It assumes that you have basic knowledge on using the command line on a remote server (bash, ssh authentification..).

The sections :ref:`Pre-setup` and :ref:`Docker-specific preparations` should be done just the first time.
Once it is done, you can directly go to the section :ref:`Spawn a BinderHub instance using Docker`.

Pre-setup
*********

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
****************************

You will install a trusted Docker image that will later be used to spawn the BinderHub instance.

1. Install `Docker <https://www.Docker.com/get-started>`_ and log in to the dockerhub with your credentials.

   .. code-block:: console

     sudo docker login

2. Pull the Docker image that encapsulates the barebones environment to spawn a BinderHub instance with our provider (compute canada as of late 2019).
   You can check the different tags available under our `dockerhub user <https://hub.Docker.com/r/conpdev/neurolibre-instance/tags>`_.

   .. code-block:: console

     sudo docker pull conpdev/neurolibre-instance:v1.2

Spawn a BinderHub instance using Docker
***************************************

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

     sudo docker stop conpdev/neurolibre-instance:v1.2
     sudo docker rm conpdev/neurolibre-instance:v1.2

Appendix A
**********

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

.. code-block:: json
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

Bare-metal to local Docker registry and volumes
-----------------------------------------------

Internet speed is the top-priority for our server.
We already experienced in the past slow internet speed on `Arbutus <https://docs.computecanada.ca/wiki/Cloud_resources#Arbutus_cloud_.28arbutus.cloud.computecanada.ca.29>`_
that caused us a lot of issues, specifically on the environment building phase.
The binderhub was stuck at the building phase, trying in vain to pull images from ``docker.io`` to our server.

.. note:: When the notebook was successfully created, slow internet is not an issue anymore because the interaction between the user and the binder instance is not demanding.

Among many `ideas <https://github.com/jupyterhub/binderhub/issues/864>`_, one of them that came up pretty quickly was to simply create our own local docker registry on arbutus. 
This would allow for low latency when pulling the images from the registry (connected to the local network where the binderhub resides).

The following documentation explains how we built our own docker registry on Arbutus, it is intended for developpers who want to spawn a new ``Binderhub`` on another ``openstack`` host.
It contains also instructions on how to create volumes on ``openstack`` (for the ``Repo2Data`` databases) and attach them to the docker registry.

.. note:: It is still not the case, but in the future we expect the docker registry spawning to be part of the terrafrom configurations.

Instance spawning
*****************

The first thing to do is to create a new instance on Arbutus using ``openstack``. 
It provides a graphical interface to interract with our openstack project from computecanada.

You will first need to `log-in into the openstack dashboard <https://arbutus.cloud.computecanada.ca/>`_.

.. note:: You can request the password to any infrastructure admin if authorized.

Now you can spawn a new instance under ``Compute/Instances`` with the ``Launch Instance`` button.

.. image:: img/launch_instance.png
  :width: 800px

A new window will appear where you can describe the instance you want, the following fields are mandatories:

* :code:`Instance Name`: name of the instance, choose whatever you want
* :code:`Source`: OS image name used by the instance, select ``*Bionic-x64*``
* :code:`Flavor`: hardware configuration of the instance, ``c8-30gb-186`` is more than enough
* :code:`Key Pair`: list of the public ssh keys that will be allowed on the server, find the one that match the binderhub you created in :doc:`INFRASTRUCTURE/Bare-metal to BinderHub`

Click on ``Launch instance`` at the bottom when you finished.

External floating IP
********************

To access the instance from the outside, we need a ``public floating IP`` pointing to the instance.
If you don't already have one, you can allocate a new IP under ``Network/Floating IPs`` and by clicking to ``Allocate IP To Project``.

When it is done, click on the right of the instance under ``Compute/Instances`` to associate this new floating IP. 

.. image:: img/instance_menu.png
  :width: 150px

.. warning:: You have a limited amount of floating IPs, so be carefull before using one.

Firewall
********

Firewall rules will help you protect the instance against intruders and can be created on ``openstack`` via ``Security Groups``.

1. Create a new ``Security Group`` under ``Network/Security Groups``.
2. Click on ``Manage rules`` on the right and create an ``IPV4`` rule for all ``IP Protocol`` and ``Port Range``, with a ``Remote CIDR`` from your local network.
   
   For example, if the internal ``IP address`` from your instances is in the range ``192.167.70.XX``, the ``Remote CIDR`` would be ``192.167.70.0/24``.

   .. note:: Using a ``Remote CIDR`` instead of ``Security Group`` could be considered as unsafe. But in our case it is the easiest way to allow access, since all our binderhub instances uses the same private network.

3. Enable also the ports ``22 (SSH)``, ``80 (HTTP)`` and ``443 (HTTPS)``.
4. Update the ``Security Group`` under ``Compute/Instances``, and click on the right to select ``Edit Security Groups``.

You should now have ``ssh`` access for the ``ubuntu`` user on the instance

.. code:: console

   ssh ubuntu@<floating_ip>

.. warning:: If you cannot access the instance at this time, you should double check the public key and/or the firewall rules.
   It is also possible you hit some limit rate from compute canada, so retry later.

DNS specific considerations
***************************

We will need to secure the Docker registry through ``HTTPS`` to use it with ``Binderhub``, `it is not possible otherwise <https://github.com/jupyterhub/binderhub/issues/992>`_.

The Cloudflare DNS will defined the registry domain and provide the ``TLS`` certificate for us.

1. Log-in to `cloudflare <https://dash.cloudflare.com/login>`_

.. note:: You can request the password to any infrastructure admin if authorized.

2. Under the ``DNS`` tab, you have the option to create a new record

   .. image:: img/dns_registry.png

3. Create an ``A`` record with a custom sub-domain, and the ``IPV4`` address pointing to the floating IP from :doc:`INFRASTRUCTURE/Local Docker registry setup/External floating IP`.

Volumes creation
****************

One feature of ``Neurolibre`` is to provide database access to the users of the ``Binderhub``, through user predefined `Repo2Data requirement file <https://github.com/SIMEXP/Repo2Data#input>`_.
These databases are stored into a specific volume on the Docker registry instance.

In the same time, another specific volume contains all the docker images for the registry.

These volumes will be created through ``openstack``.

1. Go under ``Volumes/Volumes`` tab
2. Click on ``Create a Volume`` and define the name of the volume and its storage size
3. Attach this volume to the Docker registry instance by clicking on the right of the instance under ``Compute/Instances``
4. Repeat the process from (1) to (3) to create the Docker registry image volume

Once the volumes are created on ``openstack``, we can ``ssh`` to the registry instance and mount the volumes:

1. Check that the volume(s) are indeed attached to the instance (should be ``/dev/vdc``):

   .. code:: console

      sudo fdisk -l

2. Now we can configure the disk to use it,

   .. code:: console

      sudo parted /dev/vdc
      mklabel gpt
      mkpart
      (enter)
      ext3
      0%
      100%
      quit

3. Check that the partition appears (should be ``/dev/vdc1``):

   .. code:: console

      sudo fdisk -l

4. Format the partition,

   .. code:: console

      sudo mkfs.ext3 /dev/vdc1

5. Create a directory and mount the partition on it:

   .. code:: console

      sudo mkdir /DATA
      sudo chmod a+rwx /DATA
      sudo mount /dev/vdc1 /DATA

6. Check if ``/dev/vdc1`` is mounted on ``/DATA``

7. Repeat all the steps from (1) to (6) for the Docker registry volume (name of directory would be ``/docker-registry``).

Docker registry setup
*********************

After ``ssh`` to the instance, install Docker on the machine by following `the official documentation <https://docs.docker.com/install/linux/docker-ce/ubuntu/#install-docker-engine---community>`_.

We will now secure the registry with a password.
Create a directory ``auth`` and a new ``user`` and ``password``:

.. code:: console

   mkdir auth
   sudo docker run --entrypoint htpasswd registry:2 -Bbn user password > auth/htpasswd

After that you can launch the registry,

.. code:: console

   sudo docker run -d -p 80:80 --restart=always --name registry \
   -v /certs:/certs \
   -v /docker-registry:/var/lib/registry \
   -v /home/ubuntu/auth:/auth -e "REGISTRY_AUTH=htpasswd" \
   -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
   -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
   -e REGISTRY_HTTP_ADDR=0.0.0.0:80 \
   registry:2

.. warning:: ``/docker-registry`` is the Docker registry volume that we configured in :doc:`INFRASTRUCTURE/Volumes creation`.

Now the registry should be running, follow `this documentation <https://docs.docker.com/registry/deploying/#copy-an-image-from-docker-hub-to-your-registry>`_ to test it.

You can try it on your machine (or another instance).
You would first need to log-in to the Docker registry using the domain name you configure ``my-binder-registry.conp.cloud`` in :doc:`INFRASTRUCTURE/DNS specific considerations`:

.. code:: console

   sudo docker login my-binder-registry.conp.cloud
   sudo docker pull ubuntu:16.04
   sudo docker tag ubuntu:16.04 my-binder-registry.conp.cloud/my-ubuntu
   sudo docker push my-binder-registry.conp.cloud/my-ubuntu

.. note:: The Docker registry can be accessed through its `HTTP api <https://docs.docker.com/registry/spec/api/>`_. This is how you can delete images from the registry for example.

BinderHub test mode
-------------------

To make changes to the K8s integration of BinderHub, such as injecting `repo2data` specific `labels` to a `build pod`, we need to bring up a BinderHub for development.
The following guidelines are inhereted from `the original BinderHub docs <https://github.com/jupyterhub/binderhub/blob/master/CONTRIBUTING.md#kubernetes-integration-changes>`_. This documentation assumes that the development
is to be done in a remote node via :code:`ssh` access.

1. :code:`ssh` into the node

2. Launch shell as the root user:

   .. code-block:: console

     sudo su - root

3. Make sure that the following :code:`apt` packages are installed
      * :code:`npm`
      * :code:`git`
      * :code:`curl`
      * :code:`python3`
      * :code:`python3-pip`
      * :code:`socat`

4. Ensure that the :code:`minikube` is installed, if not `follow these instructions <https://kubernetes.io/docs/tasks/tools/install-minikube/>`_.

5. Clone the :code:`BinderHub` repo and :code:`cd` into it:

   .. code-block:: console

     git clone https://github.com/jupyterhub/binderhub
     cd binderhub

6. Start :code:`minikube`:

   .. code-block:: console

     minikube start

7. Install :code:`helm` to the minikube cluster:

   .. code-block:: console

     curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get | bash

8. Initialize :code:`helm` in the minikube cluster:

   .. code-block:: console

     helm init

9. Add :code:`JupyterHub` to the helm charts:

   .. code-block:: console

     helm repo add jupyterhub https://jupyterhub.github.io/helm-chart/
     helm repo update

   The process is successfull if you see the :code:`Hub is up` message.

10. Install BinderHub and its development requirements:

   .. code-block:: console

     python3 -m pip install -e . -r dev-requirements.txt

11. Install JupyterHub in the minikube with helm:

   .. code-block:: console

     ./testing/minikube/install-hub

12. Make minikube use the host Docker daemon :

   .. code-block:: console

     eval $(minikube docker-env)

   Expect :code:`'none' driver does not support 'minikube docker-env' command` message.
   This is intended behavior.

13. Run :code:`helm list` command to see if the JupytherHub is listed. It should look   like:

   .. code-block:: console

     binder-test-hub 1 DEPLOYED jupyterhub-0.9.0-beta.4 1.1.0

Now, you are ready to start BinderHub with a config file. As done in the reference doc,
start the binderhub with the config in the :code:`testing` directory:

   .. code-block:: console

     python3 -m binderhub -f testing/minikube/binderhub_config.py

Note that you are starting :code:`BinderHub` with module name. This is possible
thanks to the step-10 above. In that step, :code:`-e` argument is passed to :code:`pip`
to point the local :code:`../binderhub` directory as the project path via :code:`.` value. This is why the changes you made in the :code:`/binderhub` directory will take effect.

There are some details worth knowing in the :code:`testing/minikube/binderhub_config.py` file, such as:

   .. code-block:: console

     c.BinderHub.hub_url = 'http://{}:30123'.format(minikube_ip)

This means that upon a successful build, the BinderHub session will be exposed
to :code:`your_minikube_IP:30123`. To find out your minikube IP, you can Simply
run `minikube ip` command.

The port number :code:`30123` is described in :code:`jupyterhub-helm-config.yaml`.

If everything went right, then you should be seeing the following message:

   .. code-block:: console

     [I 200318 23:53:33 app:692] BinderHub starting on port 8585

Just leave this terminal window as is. Open a new terminal and do ssh forward
the port :code:`8585` to the port :code:`4000` of your own computer by:

   .. code-block:: console

     ssh -L 4000:127.0.0.1:8585 ubuntu@<floating-ip-to-the-node>

Open your web browser and visit :code:`http://localhost:4000/`. BinderHub should
be running here.

When you start a build project by pointing BinderHub to a GitHub repo, a pod
will be associated with the process. You can see this pod by opening a `third`
terminal in your computer. Do not login shell as root in the second terminal,
which is used for :code:`ssh 8585-->4000` port forwarding.

In the 3rd terminal, do the steps 1 and 2 (above), then:

   .. code-block:: console

     kubectl get pods -n binder-test

If you injected some metadata, label etc. to a pod, you can see by:

   .. code-block:: console

     kubectl get describe -n binder-test <pod_name>

It is expected that you'll receive a 404 response after a successful Binder build.
This is because the user is automatically redirected from :code:`8585` to the instance served at :code:`your_minikube_IP:30123`.

If you would like to interact with a built environment, you need to
forward :code:`your_minikube_IP:30123` to another port in your laptop
using another terminal.

Finally, Docker images created by Binder builds in the minikube host
can be seen simply by :code:`docker images`. If you'd like to switch docker
environment back to the default user, run :code:`eval $(docker-env -u)`.

Terminate the BinderHub running on port :code:`8585` by simply `ctrl+c`.

To delete the JupyterHub running on minikube, first :code:`helm list`, then
:code:`helm delete --purge <whatever_the_name_is>`.

Further tips such as using a local :code:`repo2docker` installation instead of
the one comes in a container, enabling debug logging (really useful) and more,
please visit the `original resource <https://github.com/jupyterhub/binderhub/blob/master/CONTRIBUTING.md#tip-use-local-repo2docker-version>`_.

To see how BinderHub automates building and publishing images for helm
charts, please visit the `chartpress <https://github.com/jupyterhub/chartpress>`_.
