BinderHub test mode
===================

This document explains how to contribute to BinderHub from a bare-metal server.
If you are a Neurolibre dev, you don't need to follow :ref:`First time setup` section,
just jump directly to :ref:`Code integration` section.

First time setup
----------------

Create an instance with openstack using bionic image, don’t forget to assign a floating IP.
After, you can ssh to this instance.

.. note:: You can find detailed instructions on how to create an openstack instance in :doc:`BAREMETAL_TO_DOCKER_REGISTRY`.

All the following should be run as root :

.. code-block:: console

    sudo su - root

Now install `docker <https://docs.docker.com/install/linux/docker-ce/ubuntu/>`_.

Install npm and other dependencies :

.. code-block:: console

    apt-get install libssl-dev libcurl4-openssl-dev python-dev python3 python3-pip curl socat
    curl -sL https://deb.nodesource.com/setup_13.x | sudo -E bash -
    apt-get install -y nodejs

Install `minikube <https://minikube.sigs.k8s.io/docs/start/linux/>`_ for a bare-metal server.

Install `kubectl <https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-linux>`_.

.. warning:: Don't forget to let ``kubectl`` run commands as your own user: :code:`sudo chown -R $USER $HOME/.kube $HOME/.minikube`.

Install binderhub repo:

.. code-block:: console

    git clone https://github.com/jupyterhub/binderhub
    cd binderhub

You can now follow the `contribution guide <https://github.com/jupyterhub/binderhub/blob/master/CONTRIBUTING.md#one-time-installation>`_ from step 3.

.. note:: Since you are in a bare-metal environment like, you don’t need to use :code:`eval $(minikube docker-env)`

You can now connect and verify the binderhub installation by accessing `http://localhost:7777/ <http://localhost:7777/>`_.

Code integration
----------------

To make changes to the K8s integration of BinderHub, such as injecting `repo2data` specific `labels` to a `build pod`, we need to bring up a BinderHub for development.

The following guidelines are inhereted from `the original BinderHub docs <https://github.com/jupyterhub/binderhub/blob/master/CONTRIBUTING.md#kubernetes-integration-changes>`_. This documentation assumes that the development
is to be done in a remote node via :code:`ssh` access.

1. :code:`ssh` into the previously configured node

.. note:: Ask any infrastructure admin for the current binderhub debug instance, if authorized.

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

.. note:: You are starting ``BinderHub`` with module name. This is possible thanks to the step-10 above.
    In that step, :code:`-e` argument is passed to :code:`pip` to point the local ``../binderhub`` 
    directory as the project path via :code:`.` value.
    This is why the changes you made in the :code:`/binderhub` directory will take effect.

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