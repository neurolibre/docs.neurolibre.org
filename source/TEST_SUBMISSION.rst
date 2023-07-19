.. admonition:: Beta Release
   :class: error

   **Please be advised that our service is currently in its beta release of development.** As a beta user, your feedback and suggestions are highly valuable in helping us identify and address any issues. 
   We kindly request your patience and understanding as we work diligently to enhance the service based on your input.

Test your NRP
================================

It is really important to first test your submission locally to alleviate further issues when deploying on Neurolibre server.
You need to make sure that:

* All the notebooks run locally with the hardware requirements from computation and data section.
* The jupyter book builds fine locally (make sure that you are not using cache files).

Test locally
::::::::::::::

Assuming that:

* you already installed all the dependencies to develop your notebooks locally 
* your preprint repository follows the NeuroLibre :doc:`SUBMISSION_STRUCTURE`.

You can easily test your preprint build locally.

1. Install Jupyter Book
:::::::::::::::::::::::

 .. code-block:: shell

    pip install jupyter-book

2. Manage your data
:::::::::::::::::::

Given the following minimalistic repository structure:

.. code-block:: shell

    .
    ├── binder
    │   ├── requirements.txt
    │   └── data_requirement.json        
    ├── content
    │   ├── _build
    │   ├── notebook.ipynb
    │   ├── _config.ym
    │   └── _toc.yml
    └── README.md

Create a directory ``data`` at the root of the repository.
Install `Repo2Data <https://github.com/SIMEXP/Repo2Data>`_ and configure the ``dst`` from the requirement file so it points to the ``data`` folder.

.. code-block:: shell

  pip install repo2data

Run ``repo2data`` inside your notebook and get the path to the data.

.. code-block:: shell

  # install the data if running locally, or points to cached data if running on neurolibre
  data_req_path = os.path.join("..", "binder", "data_requirement.json")
  # download data
  repo2data = Repo2Data(data_req_path)
  data_path = repo2data.install()

.. seealso:: Check this `example for running repo2data <https://github.com/neurolibre/repo2data-caching>`_, agnostic to server data path.

3. Book build
:::::::::::::

- Navigate to the repository location in a terminal

 .. code-block:: shell

    cd /your/repo/directory

- Trigger a jupyter book build

 .. code-block:: shell

    jupyter-book build ./content

.. seealso:: Please visit reference `documentation <https://jupyterbook.org/content/execute.html?highlight=execute#execute-and-cache-your-pages>`_ on executing and caching your outputs during a book build.

Testing on NeuroLibre servers
:::::::::::::::::::::::::::::

Meet ``RoboNeuro``! Our preprint submission bot is at your service 24/7 to help you create a NeuroLibre preprint.

.. image:: https://github.com/neurolibre/brand/blob/main/png/preview_magn.png?raw=true
  :width: 500
  :align: center

We would like to ensure that all the submissions we receive meet certain requirements. To that end, we created the `RoboNeuro preview service <https://robo.neurolibre.org>`_, 
where you point RoboNeuro to a public GitHub repository, enter your email address, then sit back and wait for the results.

.. note:: RoboNeuro book build process has two stages. First, **it creates a virtual environment** based on your runtime descriptions. If this stage is successful, then it proceeds to 
          build a Jupyter Book by **re-executing your code** in that environment. 

- On a successful book build, we will return you a preprint that is beyond PDF!
- If the build fails, we will send two log files for your inspection.

.. warning:: **A successful book build on RoboNeuro preview service is a prerequisite for submission.**

            Please note that RoboNeuro book preview is provided as a public service with limited computational resources. Therefore we encourage you to build your book locally before
            requesting our online service. Instructions are available in :doc:`LOCAL_TESTING`.

Debugging for long NeuroLibre submission
----------------------------------------

As for ``mybinder``, we also provide a binder submission page so you can play with your notebooks on our servers.
Our binder submission page is available here: https://test.conp.cloud.

When this process is really useful for debugging your submission live, it can be very long to get it.
Indeed, a jupyter book build will always occur under the hood, and as part of the build process it will try to execute everything within your submission.
This can make the build process very long (especially if you have a lot of long-running notebooks), and so you will end up waiting forever to get the binder instance.

If you are in a case where the jupyter book build fails on Neurolibre for whatever reason but works locally,
you can bypass the jupyter book build to get the interactive session almost instantly.

.. note:: For example if you have “out of memory” errors on Neurolibre, you can reduce the RAM requirements on the interactive session, and try to re-run the jupyter book build directly on the fly.

Just add ``--neurolibre-debug`` in your latest commit message to bypass the jupyter book build (as in `this git commit <https://github.com/ltetrel/nimare-paper/commit/4d5938819ad0a21365bc849ab91d29211556c77d>`_).
Now if you register your repository on https://test.conp.cloud, you will have your binder instance almost instantly.
You should be able to open a terminal session or play with the notebooks from there.

.. note:: This setup requires a previous valid binder build. If you are not able to build your binder, then you don't have a choice to fix the installation locally on your PC.

.. warning:: Please remember to remove the flag ``--neurolibre-debug`` when you are ready to submit, since NeuroLibre needs to build the jupyter book.