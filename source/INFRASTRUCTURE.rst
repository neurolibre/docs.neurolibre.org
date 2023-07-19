Infrastructure overview
=======================

At the bottom of our infrastructure, we rely on `openstack <https://www.openstack.org/>`_ which spawns our multiple VMs (what we will reffer later as instance) and virtual volumes.
After successful spawning of the instance, it is assigned a floating IP used to connect to it from the outside world.
The `cloudflare DNS <https://www.cloudflare.com/>`_ then properly configure the chosen domain name under :code:`*.conp.cloud` automatically pointing to the assigned floating IP.
When the network has been properly setup, the installation can continue with `kubernetes <https://kubernetes.io/>`_ and finishes with `BinderHub  <https://BinderHub .readthedocs.io/en/latest/index.html>`_.

We want to share our experience with the community, hence all our installation scripts are open-source available under
`neurolibre/kubeadm-boostrap <https://github.com/neurolibre/kubeadm-bootstrap>`_ and `neurolibre/terraform-binderhub <https://github.com/neurolibre/terraform-binderhub>`_.

.. warning:: NeuroLibre is still at an alpha stage of development, the github repositories will change frequently so be carefull if you use them.

You can find more details on the installation at :doc:`BAREMETAL_TO_BINDERHUB`.