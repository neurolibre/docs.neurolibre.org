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

BinderHub test mode 
----------

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


