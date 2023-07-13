
Step 1 - 🍛 Prepare submission
==============================

Before submitting to Neurolibre there are several prerequisites that you must complete, those include:

.. raw:: html

    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<input type="checkbox"> <a href="#hosting">Hosting</a><br>
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<input type="checkbox"> <a href="#coding-language">Coding language</a><br>
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<input type="checkbox"> <a href="#repository-layout">Repository layout</a><br>
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<input type="checkbox"> <a href="#computation-and-data-requirements">Computation and data requirements</a><br>

------------

🛰️ Hosting
::::::::::

Github is our main hosting service to store, read and interact with any public submission.
The submission, which is in fact a github repository, will need to be public.

Where our backend supports other git hosting services, our submission workflow doesn't.
Still if you are elligible, you may use `any of those <https://binderhub.readthedocs.io/en/latest/developer/repoproviders.html#supported-repoproviders>`_.

.. warning:: We are not supporting private repositories at the moment.

👩‍💻 Coding language
::::::::::::::::::

Neurolibre is living under the jupyter notebook ecosystem, hence supporting a broad range of different coding languages.
If the one you are using is inside `the following list <https://github.com/jupyter/jupyter/wiki/Jupyter-kernels>`_, then you should be ready to go.
But please be mindful that the administrator and reviewer team mainly focuses on **python 3**, if you are using another language don't expect fine support for debugging.

We only accept submissions which are in notebook ``.ipynb`` format. Though less recommended, we also support text notebook representation (like markdown)
using `jupytext <https://jupytext.readthedocs.io/en/latest/formats.html#notebook-formats>`_.

Check the following minimalistic example of a jupyter notebook.

.. raw:: html

    <center>
      <iframe id="binder-frame" width="90%" height="600" marginwidth="5%"
      src="https://nbviewer.org/github/neurolibre/repo2data-caching/blob/master/notebooks/nilearn-example.ipynb"></iframe>
    </center>
    <br>

🗂 Repository layout
::::::::::::::::::::

Our infrastructure heavily relies on `binderhub <https://binderhub.readthedocs.io/en/latest/>`_ for the interactive compute environment, and 
`jupyter book <https://jupyterbook.org/intro.html>`_ for the publishing pdf artifact.
The backend workflow make use of those dependencies to build your submission and make the data accessible.

We recommend you to use `our template repository <SUBMISSION_STRUCTURE.html#quickstart-preprint-templates>`_ to give you a head-start.
If you prefer to build the submission yourself, you will need to check the specific detailed section on the :doc:`SUBMISSION_STRUCTURE`.

Either way you need to make sure that you have all the following files ready:

.. raw:: html

    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<input type="checkbox"> <a href="https://mybinder.readthedocs.io/en/latest/using/config_files.html#requirements-txt-install-a-python-environment">Binder requirement file</a> - The recipee that describes the software dependencies.<br>
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<input type="checkbox"> (if applicable) <a href="https://github.com/SIMEXP/Repo2Data#input">Data requirement file</a> - A file that describes the data.<br>
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<input type="checkbox"> Jupyter book <a href="https://jupyterbook.org/customize/config.html">configuration</a> and <a href="https://jupyterbook.org/structure/toc.html">table of content</a> -  To configure the jupyter book build/layout.<br>
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<input type="checkbox"> <a href="https://github.com/neurolibre/repo2data-caching/blob/master/notebooks/nilearn-example.ipynb">Notebook files</a> - Your collection of executable scripts.<br>
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<input type="checkbox"> <a href="https://joss.readthedocs.io/en/latest/submitting.html#example-paper-and-bibliography">Static summary</a> - Submission summary document and bibliography for metadata collection.<br><br>


.. seealso:: Take a look at our meta anlaysis on myelin submission which is publically available `in this github repository <https://github.com/Notebook-Factory/myelin-meta-analysis>`_.

💽 Computation and data requirements
::::::::::::::::::::::::::::::::::::

NeuroLibre strives to provides the best experience by giving access to lot of ressources, still we have limited hardware.
Taking ressources constraints into account, each submission should:

* not take more than 8 hours to execute uncached (that includes all notebooks and book build)
* run with 1 or 2 CPU@3GHz
* allocate strictly less than ~7.5GB of RAM
* have few dependencies to keep your installation light (idealy less than 1GB) and fast (<10min)
* take less than 10GB runtime storage (files generated by your scripts)
* use no more than 5GB of data (inputs, cached content), downloadable from a trusted source

.. warning::  A trusted web source is a well known dataset collection (like `openneuro <https://openneuro.org/>`_)
  or dataset fetching functions from libraries (for ex. `nilearn.datasets.fetch* <https://nilearn.github.io/modules/reference.html#module-nilearn.datasets>`_).

For additional information, our test server (to test and build submissions) uses `Intel Xeon Gold 6248 <https://ark.intel.com/content/www/us/en/ark/products/192446/intel-xeon-gold-6248-processor-27-5m-cache-2-50-ghz.html>`_,
while the production server (where the final submission lives) has `Intel Xeon E5-2650 <https://ark.intel.com/content/www/us/en/ark/products/64590/intel-xeon-processor-e52650-20m-cache-2-00-ghz-8-00-gts-intel-qpi.html>`_.
