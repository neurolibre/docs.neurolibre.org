.. warning:: NeuroLibre is at an alpha stage of development, and is not currently open for submissions.

Preprint repository structure
================================

We expect to find all the submission material in a **public** GitHub repository that has the following structure:

| neurolibre_submission
| .
| ├── binder
| │   ├── requirements.txt
| │   └── (data_requirement.json)
| │
| ├── content
| │   ├── _toc.yml
| │   ├── _config.yml
| │   ├── 01-simple_notebook.ipynb
| │   └── (...)
| ├── paper.md
| └── paper.bib

  .. warning:: If RoboNeuro does not see this file layout, it will fail to build the jupyter book build (but may be able to build the computing environment).
              Make sure that your file layout never change during runtime (especially if using a Dockerfile).

⏩ Quickstart: Preprint templates
:::::::::::::::::::::::::::::::::

To give you a head-start, we created preprint template repositories:

.. list-table::
   :widths: 33 33 33
   :header-rows: 1

   * - Preprint
     - Programming language
     - GitHub repository
   * - Link
     - Python
     - `neurolibre/template <https://github.com/neurolibre/template>`_
   * - Link
     - C++
     - `neurolibre/cpp <https://github.com/neurolibre/binder-cpp>`_

1. Choose your template from the list below and create a new repository into your (or to an organization) account.
2. Follow the instructions in the ``README`` file.
3. Test your book build using `RoboNeuro preview service <https://roboneuro.herokuapp.com>`_.

The following section provides further detail about the structure of a NeuroLibre preprint repository. 

1. The ``binder`` folder
::::::::::::::::::::::::

.. image:: https://github.com/neurolibre/brand/blob/main/png/binder_folder.png?raw=true
  :width: 800
  :align: left
                  

1.1 Runtime
-----------

Preprint-specific runtime dependencies
......................................

  The execution runtime can be based on any of the (non-proprietary) programming languages supported by Jupyter. NeuroLibre looks at the
  ``binder`` folder to find some configuration files such as a ``requirements.txt`` (Python), ``R.install`` (R), ``Project.toml`` (Julia)
  or a ``Dockerfile``.

.. seealso:: 
  The full list of supported configuration files is available `here <https://mybinder.readthedocs.io/en/latest/using/config_files.html>`_.

Environment configuration for NeuroLibre
........................................

  You should try to make your environment clean and concize, that is why the prefered configuration file for NeuroLibre are the
  ``requirements.txt``.

  It should be small (to keep environment building and loading as short as possible), and versionnized (so your
  environment is fully reproducible, and cache-able).

  For example this requirement is bad because it has lot of unnecessary dependencies:

  .. code-block:: text

    numpy
    scipy
    jupyter
    matplotlib
    Pillow
    scikit-learn
    tensorflow

  On the other hand, this one is concise, reproducible and will take much less time to build:

  .. code-block:: text

    tensorflow==2.4.0

  .. warning:: Starting from ``pip 20.3``, `the package resolver changed its behaviour <https://pip.pypa.io/en/stable/user_guide/#changes-to-the-pip-dependency-resolver-in-20-3-2020>`_ to reduce inconsistencies in software versions.
            As a consequence and if your submission has lot of interdependent dependencies, your build may a while.
            This is typically the case if you see messages like this during the build:
              .. code-block:: text

                INFO: pip is looking at multiple versions of linkify-it-py to determine which version is compatible with other requirements. This could take a while.

NeuroLibre dependencies
.......................

  Our test server creates a virtual environment in which your content is re-executed to build a Jupyter Book. To enable this, we need some 
  Python packages.

  - If you are using configuration files, we need the following in a ``requirements.txt`` file:

  .. code-block:: text

    jupyter-book
    jupytext

  - If your environment is described by a ``Dockerfile`` you can use our base image: 

  .. code-block:: docker
    :emphasize-lines: 1

    FROM neurolibre/book:latest
    ...
    
  .. warning:: Make sure that your whole environment is not too big (>1GB of installed dependencies), especially if you are using a `Dockerfile`.
      Large environments increase the binder spawn time, impact your computing performance, and takes a lot of space on our servers.

1.2 Data
--------

NeuroLibre offers generous data storage and caching to supercharge your preprint. If your executable content consumes input data, you need to read this section carefully. Indeed, we don't allow data download other than through our method.

To download data, NeuroLibre looks for a `repo2data <https://github.com/SIMEXP/Repo2Data>`_ configuration file: ``data_requirement.json``.
This file must point to a **publicly available dataset**, so it can be available during preprint runtime.

.. seealso:: **Repo2data** can download data from several resources including OSF, datalad, zenodo or aws. For details, please visit `the documentation <https://github.com/SIMEXP/Repo2Data>`_.

Example preprint templates using ``repo2data`` for caching data on NeuroLibre servers:

.. list-table::
   :widths: 50 50
   :header-rows: 1

   * - Download Resource
     - GitHub repository
   * - Nilearn
     - `neurolibre/repo2data-nilearn <https://github.com/neurolibre/repo2data-caching>`_
   * - OSF
     - `neurolibre/repo2data-osf <https://github.com/neurolibre/neurolibre-osf-test>`_

.. warning:: 
  RoboNeuro may fail downloading relatively large datasets (**exceeding 1GB**) or if the data server is to slow, as the book build process times out after 10 minutes. This is because of some limitations, independent from us, in our software stack.
  If you face some problems when downloading your data, please create an issue in your github repository so a Neurolibre admin can check it.

.. topic:: Help RoboNeuro find your data during book build

  `Repo2Data <https://github.com/SIMEXP/Repo2Data>`_ downloads your data to a folder named ``data``, which is created at the base of your repository.

  .. note:: We suggest using repo2data locally before you request a RoboNeuro preview service.
    Matching `this data loading convention <#testing-book-build-locally>`_ will increase your chances of having a successful NeuroLibre preprint build, and will make
    your data dependency agnostic to computer.

  Assuming you are running a notebook on NeuroLibre and have a requirement file as:

  .. code-block:: bash

    { "src": "download_my_brain(data_dir=_dst);",
    "projectName": "PROJECT_NAME"}


  - A code cell in a ``content/my_notebook.ipynb`` would access data by:

    .. code-block:: python

      import nibabel as nib
      import os
      img = nib.load(os.path.join('..', 'data', 'PROJECT_NAME', 'my_brain.nii.gz'))

  - A code cell in a ``content/01/my_01_notebook.ipynb`` would access data by:

    .. code-block:: python

      import nibabel as nib
      img = nib.load(os.path.join('..', '..', 'data', 'PROJECT_NAME', 'my_brain.nii.gz')) # In this case, 2 upper directories

  If the data directories in your code cells are not following this convention, RoboNeuro will fail to re-execute your notebooks and interrupt the book build.

.. warning:: If you are a Windows user, manually defined paths (e.g. ``.\data\my_data.txt``) won't be recognized by the preprint runtime.
             Please use an operating system agnostic convention to define paths, like ``os.path.join`` in Python.
        
2. The ``content`` folder
:::::::::::::::::::::::::

.. image:: https://github.com/neurolibre/brand/blob/main/png/content_folder.png?raw=true
  :width: 800
  :align: left

2.1 Executable & narrative content
----------------------------------

NeuroLibre accepts the following file types to create a preprint that is beyond PDF:

- ✅ Jupyter Notebooks, 
- ✅ `MyST <https://github.com/neurolibre/template/blob/main/content/02-simple-myst.md>`_ formatted markdown.
- ✅ Plain text markdown files.
- ✅ A mixture of all above

.. warning:: ❌  We don't accept markdown files with narrative content **only**, that is not really beyond PDF :)

.. note:: ✅  You can organize your content in sub-folders.

.. topic:: Writing narrative content
   
   Jupyter Book provides you with an arsenal of authoring tools to include citations, equations, figures, special content
   blocks and more into your notebooks or markdown files.
  
  .. seealso:: Please visit the corresponding Jupyter Book `documentation page <https://jupyterbook.org/content/index.html#write-narrative-content>`_ for guidelines.

.. topic:: Writing executable content

   Based on the powerful Jupyter ecosystem, NeuroLibre preprints allow you to interleave computational material with your narrative.
   You can add some directives and metadata to your code cell blocks for Jupyter Book to determine the format and behavior of the outputs,
   such as interactive data visualization.

  .. seealso:: Please visit the corresponding Jupyter Book `documentation page <https://jupyterbook.org/execute/index.html#write-executable-content>`_ for guidelines.

There are two **mandatory** files that we look for in the ``content`` folder: ``_config.yml`` and ``_toc.yml``. These files 
help RoboNeuro structure your book and configure some settings.

2.2 Table of contents
---------------------

The ``_toc.yml`` file determines the structure of your NeuroLibre preprint. It is a simple configuration file 
specifying a table of content from all the executable & narrative content found in the ``content`` folder (and in subfolders).

.. seealso:: The complete reference for the ``_toc.yml`` can be found `here <https://jupyterbook.org/customize/toc.html>`_.

2.3 Book configuration
----------------------

The ``_config.yml`` file governs all the configuration options for your Jupyter Book formatted preprint, such as adding a logo, 
enable/disable interactive buttons or control notebook execution and caching settings. Few important points:

- Please ensure that the title and the list of authors matches those specified in the ``paper.md``.

 .. code-block:: yaml

   title:  "NeuroLibre preprint template"  # Add your title
   author: John Doe, Jane Doe  # Add author names

- Please ensure that the repository address is accurate.

 .. code-block:: yaml

   repository:
     url: https://github.com/username/reponame  # The URL to your repository

.. seealso:: The complete reference for the ``_config.yml`` can be found `here <https://jupyterbook.org/customize/config.html>`_.

3. Static summary
:::::::::::::::::

.. image:: https://github.com/neurolibre/brand/blob/main/png/paper.png?raw=true
  :width: 800
  :align: left

The front matter of ``paper.md`` is used to collect meta-information about your preprint:

.. code-block:: yaml

  ---
  title: 'White matter integrity of developing brain in everlasting childhood'
  tags:
    - Tag1
    - Tag2
  authors:
    - name: Peter Pan
      orcid: 0000-0000-0000-0000
      affiliation: "1, 2"
    - name: Tinker Bell
      affiliation: 2
  affiliations:
  - name: Fairy dust research lab, Everyoung state university, Nevermind, Neverland
    index: 1
  - name: Captain Hook's lantern, Pirate academy, Nevermind, Neverland
    index: 2
  date: 08 September 1991
  bibliography: paper.bib
  ---

The corpus of this static document is intended for a big picture summary of the preprint
generated by the executable and narrative content you provided (in the ``content``) folder. You can include citations
to this document from an accompanying BibTex bibliography file ``paper.bib``.

To check if your PDF compiles, visit RoboNeuro `preprint preview page <https://roboneuro.herokuapp.com>`_, select `NeuroLibre PDF` option and enter your repository address.

.. seealso:: For more information on how to format your paper, please `take a look at JOSS documentation <https://joss.readthedocs.io/en/latest/submitting.html#example-paper-and-bibliography>`_.
