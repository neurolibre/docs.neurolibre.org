1. Prepare submission
=====================

Before submitting to Neurolibre there are several prerequisites that you must complete, those include:

.. raw:: html

    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<input type="checkbox"> <a href="#hosting">Hosting</a><br>
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<input type="checkbox"> <a href="#coding-language">Coding language</a><br>
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<input type="checkbox"> <a href="#computation-and-data-requirements">Computation and data requirements</a><br>
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<input type="checkbox"> <a href="#repository-layout">Repository layout</a><br>
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<input type="checkbox"> <a href="#local-testing">Local testing</a><br><br>


Hosting
:::::::

Github is our main hosting service to store, read and interact with any public submission.
The submission, which is in fact a github repository, will need to be public.

Where our backend supports other git hosting services, our submission workflow doesn't.
Still if you are elligible, you may use `any of those <https://binderhub.readthedocs.io/en/latest/developer/repoproviders.html#supported-repoproviders>`_.

.. note:: We are not supporint private repositories at the moment.

Coding language
:::::::::::::::

Neurolibre is living under the jupyter notebook ecosystem, hence supporting a broad range of different coding languages.
If the one you are using is inside `the following list <https://github.com/jupyter/jupyter/wiki/Jupyter-kernels>`_, then you should be ready to go.
But please be mindful that the administrator and reviewer team mainly focuses on python 3, if you are using another language don't expect fine support for debugging.

We only accept submissions which are in notebook ``.ipynb`` format. Though less recommended, we also support text notebook representation (like markdown)
using `jupytext <https://jupytext.readthedocs.io/en/latest/formats.html#notebook-formats>`_.

Check the following minimalistic example of a jupyter notebook.

.. raw:: html

    <center>
      <iframe id="binder-frame" width="90%" height="600" marginwidth="5%"
      src="https://nbviewer.org/github/neurolibre/repo2data-caching/blob/master/notebooks/nilearn-example.ipynb"></iframe>
    </center>
    <br>

Computation and data requirements
:::::::::::::::::::::::::::::::::

NeuroLibre strives to provides the best experience by giving access to lot of ressources, still we have limited hardware.
Taking ressources constrains into account, and each submission should:

* not take more than 8hours to execute uncached (that includes all notebooks and build)
* use 1 or 2 CPU@3GHz
* allocate strictly less than ~7.5GB of RAM
* take less than 10GB storage (for all generated files)
* use no more than 5GB for input datasets and downloadable publically from a trusted source.

.. warning::  A trusted source is a dataset that is hosted on a well known dataset collection (like `openneuro <https://openneuro.org/>`_)
  or dataset fetching function from softwares (for ex. `nilearn.datasets.fetch* <https://nilearn.github.io/modules/reference.html#module-nilearn.datasets)>`_).

For additional information, our test cluster (to test their submission) has an Intel Xeon Gold 6248, while the production cluster (where the final submission lives) has an Intel Xeon E5-2650.


Repository layout
:::::::::::::::::

(link to repository structure with all informations)

Local testing
:::::::::::::

Additionnally: (link to local testing with jupyter book)