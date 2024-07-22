FAQ - Frequently Asked Questions
================================

.. contents::
    :local:
    :depth: 1

------------

How can I test a NeuroLibre submission ?
::::::::::::::::::::::::::::::::::::::::
You can test your NeuroLibre submission using our `RoboNeuro preview service <https://roboneuro.herokuapp.com>`_.
Make sure to follow :doc:`TEST_SUBMISSION` if you need more details.

Can I submit a non-github repository ?
::::::::::::::::::::::::::::::::::::::
We don't accept non-github submission.
Still if you need the interactive binder, you can use any of `those providers <https://binderhub.readthedocs.io/en/latest/developer/repoproviders.html#supported-repoproviders>`_.

What are the hardware limits on NeuroLibre ?
::::::::::::::::::::::::::::::::::::::::::::
Running time should take less than 8 hours to execute uncached (that includes all notebooks and book build) with  or 2 CPU@3GHz.
You need use less than ~7.5GB of RAM, take less than 10GB runtime storage and no more than 5GB of data.

I want to contribute, how can I do that ?
:::::::::::::::::::::::::::::::::::::::::
It would be a pleasure to include external people on our project, please reach out to us via our `mattermost brainhack forum <https://mattermost.brainhack.org>`_ or #TODO:EMAIL!
There are fundamentally two ways to contribute to NeuroLibre: as a `reviewer <REVIEWER.html>`_ or as a `developer <INFRASTRUCTURE.html>`_.

The reviewer team is in charge of checking if submissions execute ok on our servers, there are also exchanging with the author to help them improve it.
Developer team works on the binderhub administration, backend workflows and frontend (including github integrations, and JOSS website template).

Which languages does NeuroLibre support ?
:::::::::::::::::::::::::::::::::::::::::
All languages supported by the jupyter ecosystem, check the `following list <https://github.com/jupyter/jupyter/wiki/Jupyter-kernels>`_.

What type of review are you doing ?
:::::::::::::::::::::::::::::::::::
We are a preprint service, and so we stand for minimalistic reviews.
This includes basic formatting, and checking if code executes.

How do I manage datasets with NeuroLibre ?
::::::::::::::::::::::::::::::::::::::::::
We use `repo2data <https://github.com/SIMEXP/Repo2Data>`_ to manage input data, and cache.
For more information please check `the data related section <SUBMISSION_STRUCTURE.html#data>`_.

Which version of ``jupyter-book`` and ``repo2data`` should I use ?
::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
We always recommend latest versions for better compatibility.
For ``jupyter-book`` you are free to use any version you like, since we rely only on the compiled artifacts.
For ``repo2data``, we highly advise to match latest version because this is what is used on the backend.

How can I cache my experiments ?
::::::::::::::::::::::::::::::::
If you have some cache data, you can use ``repo2data`` to make the data available on NeuroLibre.
Then follow the information about jupter-book caching `here <https://jupyterbook.org/content/execute.html?highlight=execute#execute-and-cache-your-pages>`_.

Can I use Dockerfile for my submission ?
::::::::::::::::::::::::::::::::::::::::
As highlghted in our documentation, we don't recommend building with Dockerfiles.
However if you don't have choice, you can check `our section <SUBMISSION_STRUCTURE.html#environment-configuration-for-neurolibre>`_,
and more specifically `binder with Dockerfile instructions <https://mybinder.readthedocs.io/en/latest/tutorials/dockerfile.html>`_.
