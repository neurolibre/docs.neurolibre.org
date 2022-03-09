Local testing
=============

Assuming that:

- you already installed all the dependencies to develop your notebooks locally 
- your preprint repository follows the `NeuroLibre preprint structure <#preprint-repository-structure>`_

you can easily test your preprint build locally.

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

1. Book build
:::::::::::::

- Navigate to the repository location in a terminal

 .. code-block:: shell

    cd /your/repo/directory

- Trigger a jupyter book build

 .. code-block:: shell

    jupyter-book build ./content

.. seealso:: Please visit reference `documentation <https://jupyterbook.org/content/execute.html?highlight=execute#execute-and-cache-your-pages>`_ on executing and caching your outputs during a book build.