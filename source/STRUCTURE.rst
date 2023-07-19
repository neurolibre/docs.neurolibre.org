.. role:: contentlabel
  :class: contentlabel

.. role:: binderunlabel
  :class: binderunlabel

.. role:: binderdatalabel
  :class: binderdatalabel

Structure your NRP repository
==============================

Scholarly publishing has evolved from the clunky days of typewriters and snail mail, to the digital age of electronic word documents.
The next step of the evolution takes root from a **GitHub repository** behold the NeuroLibre Reproducible Preprints (NRP)!

.. image:: https://github.com/neurolibre/brand/blob/main/png/nrp_init.png?raw=true
     :align: center
     :width: 800

The illustration above is a concise overview of the key components required to bring an NRP to life from a public GitHub repository.

The following sections provide details on the expected layout of an NRP repository that lives on GitHub.

🟠 :contentlabel:`The content folder`
::::::::::::::::::

To provide a powerful, flexible, and interactive way to create your preprint, NRPs are based on `the Jupyter Book <https://jupyterbook.org>`_.

.. image:: https://github.com/neurolibre/brand/blob/main/gif/content_interact.gif?raw=true
     :align: center
     :width: 500

When building the Jupyter Book for an NRP (which is a compact website), NeuroLibre expects locating your **Jupyter Notebooks** and/or **MyST Markdown** files
within a folder named :contentlabel:`content`.

.. sidebar:: 📑 Reference documentations

  .. raw:: html
  

   <ul>
   <li><a href="https://jupyterbook.org/en/stable/content/myst.html" target="_blank">Writing narrative content</a></li>
   <li><a href="https://jupyterbook.org/en/stable/content/execute.html" target="_blank">Writing executable content</a></li>
   <li><a href="https://jupyterbook.org/en/stable/file-types/index.html" target="_blank">Allowed content types</a></li>
   <li><a href="https://jupyterbook.org/en/stable/content/citations.html" target="_blank">Citations and bibliography</a></li>
   <li><a href="https://jupyterbook.org/en/stable/interactive/interactive.html" target="_blank">Interactive outputs</a></li>
   <li><a href="https://jupyterbook.org/en/stable/structure/toc.html" target="_blank"><code>_toc.yml</code></a></li>
   <li><a href="https://jupyterbook.org/en/stable/structure/config.html" target="_blank"><code>_config.yml</code></a></li>
   </ul>

Inside the :contentlabel:`content` directory, you have the freedom to organize the ``SOURCE`` files as per your preference::

    root/
    ├─ content/
    │  ├─ _toc.yml                  [REQUIRED]
    │  ├─ _config.yml               [REQUIRED]
    │  ├─ _neurolibre.yml           [OPTIONAL]
    │  ├─ my_notebook.ipynb         [SOURCE]
    │  ├─ my_myst.md                [SOURCE]
    │  ├─ MY FOLDER
    │  │  ├─ another_notebook.ipynb [SOURCE]

|
ℹ️ The relationship between the source files and the table of contents of your NRP must be defined in the :kbd:`content/_toc.yml` file, as it is a 
``REQUIRED`` component.

ℹ️ Another ``REQUIRED`` component is the  :kbd:`content/_config.yml` to customize the appearance and behavior of your Jupyter Book.

.. admonition:: 💻 Supported programming languages
   :class: dropdown contentad

   NRPs, being part of the Jupyter ecosystem, offer the flexibility to utilize a wide range of programming languages, 
   provided they do not require a license (e.g., MATLAB is not supported yet, but you can use Octave).

   You can take advantage of any language that has a compatible  kernel listed in the `Jupyter kernels <https://github.com/jupyter/jupyter/wiki/Jupyter-kernels>`_ for writing the 
   executable content of your NRP.

   Another important consideration is to ensure that `BinderHub configurations <https://mybinder.readthedocs.io/en/latest/examples/sample_repos.html>`_ support the language
   of your choice, or you know how to create a Dockerfile to establish a reproducible runtime environment. Further detail on this matter is provided in the 
   following (green) section.

.. admonition:: 📝 Managing citations and bibliography in your reproducible preprint
   :class: dropdown contentad

   To cite articles in your reproducible preprint, include your bibtex formatted bibliographic entries in a :kbd:`paper.bib` file located at the root of your repository,
   which is the same bibliography used by the companion PDF. 
   
   To point the Jupyter Book build to the relevant bibliography, add the following to the :kbd:`content/_config.yml` file: 

   .. code-block:: yaml

      bibtex_bibfiles:
        - ../paper.bib
      sphinx:
        config:
          bibtex_reference_style: author_year
    
    For further details regarding the management of citations and bibliography in Jupyter Book, please see the `reference docs <https://jupyterbook.org/en/stable/content/citations.html>`_.

.. admonition:: ➕ Reproducible preprint in disguise (traditional article layout)
   :class: dropdown contentad

   If you prefer your reproducible preprint to have a layout resembling a traditional article — single page and without sidebars — 
   you can achieve this by creating your content in a single Notebook or MyST markdown file. Additionally, include a :kbd:`content/_neurolibre.yml` 
   file with the following content:

   .. code-block:: yaml

     book_layout: traditional
     single_page: index.ipynb
  
   Below is an example of an NRP that combines the appearance of a traditional article with the powerful features of a Jupyter Book:
  
   .. raw:: html

    <center>
      <iframe id="binder-frame" width="90%" height="600" marginwidth="5%"
      src="https://preview.neurolibre.org/book-artifacts/rrsg2020/github.com/paper/562fb392105e48b5da7a4b6bfcf82d7504ea7c71/_build/_page/index/singlehtml"></iframe>
    </center>
    <br>

.. admonition:: 🎚 Make the most of your NRP with interactive visualizations
   :class: contentad

   We strongly recommend incorporating interactive visualizations, such as those offered by `plotly <https://plotly.com/>`_, to enhance the value of your NRP.

   By utilizing interactive visualizations, you can fully leverage the potential of your figures and present your data in a more engaging and insightful manner.
   
   You can visit the `reference JupyterBook documentation <https://jupyterbook.org/en/stable/interactive/interactive.html?highlight=interactive>`_ to have your interactive outputs rendered in your NRP.  


🟢 :binderunlabel:`The binder folder (runtime)`
::::::::::::::::::

One of the essential features of NRPs is the provision of dedicated BinderHub instances for the published preprints. 
This empowers readers and researchers to interactively explore and reproduce the findings presented in the NRP through 
a web browser, without installing anything to their computers.

.. image:: https://github.com/neurolibre/brand/blob/main/gif/binder_folder.gif?raw=true
     :align: center
     :width: 500

By leveraging NeuroLibre's BinderHub, each NRP receives its isolated computing environment, ensuring that the code, data, and 
interactive elements remain fully functional and accessible. 

The NRP repository's :binderunlabel:`binder` folder contains all the essential runtime descriptions to tailor such isolated computing
environments for each reproducible preprint.

.. admonition:: ⚙️ How to setup your runtime
   :class: binderun
   
   To specify your runtime and set up the necessary configuration files for your runtime environment, please refer to the `binderhub configuration files documentation <https://mybinder.readthedocs.io/en/latest/using/config_files.html>`_. 
   
   To implement this in your NRP repository, create a :binderunlabel:`binder` folder and place the appropriate configuration files inside it according to your runtime requirements. These configuration files will define the environment in which your preprint's code 
   and interactive elements will run when accessed through NeuroLibre's BinderHub.

.. admonition:: ⚠️ NeuroLibre specific dependencies
   :class: binderun

   As we build a Jupyter Book for your NRP in the exact same runtime you defined, we need the following Python dependencies 
   to be present. For example, in a :kbd:`binder/_requirements.txt` file::

      repo2data>=2.6.0
      jupyter-book==0.14.0

   ❗️We recommend not using ``jupyter-book`` versions newer than ``0.14.0`` as of July 2023.

   Currently, we are using ``repo2data`` to download the dataset needed to run your executable content. For details, please see the following (blue) section.

.. admonition:: 🔗 Example runtime environments
   :class: dropdown binderun

   You can explore the `binder-examples GitHub organization <https://github.com/binder-examples>`_ to find useful examples of configuration files.

   Moreover, for additional insights and inspiration, you can visit the `roboneurolibre GitHub organization <https://github.com/roboneurolibre>`_ to explore various NRP repositories. 
   Observe how each preprint defines its runtimes and customizes the Binder environment to suit their research needs.

.. admonition:: 🔋  Ensuring reproducibility and resource allocation in NRPs
   :class: binderun

   As of July 2023, each NRP Jupyter Book build is allocated the following resources:

   * 8 hours of execution time 
   * 1 or 2 CPUs at 3GHz
   * 6GB of RAM

   Please note that the Jupyter Book build (``book build``) occurs only after a successful ``runtime build`` (BinderHub). 
   The resource allocations mentioned above apply specifically to the ``book build``.
   
   Understanding the distinction between the ``runtime build`` and ``book build`` is crucial for adhering to reproducible practices. 
   
   **It is strongly advised NOT to download external dependencies during the book build**, as NeuroLibre cannot guarantee their long-term preservation. 
   As a best practice, all runtime dependencies should be handled during the ``runtime build`` using the BinderHub configuration files.


🔵 :binderdatalabel:`The binder folder (data)`
::::::::::::::::::


* take less than 10GB runtime storage (files generated by your scripts)
* use no more than 5GB of data (inputs, cached content), downloadable from a trusted source

.. warning::  A trusted web source is a well known dataset collection (like `openneuro <https://openneuro.org/>`_)
  or dataset fetching functions from libraries (for ex. `nilearn.datasets.fetch* <https://nilearn.github.io/modules/reference.html#module-nilearn.datasets>`_).

For additional information, our test server (to test and build submissions) uses `Intel Xeon Gold 6248 <https://ark.intel.com/content/www/us/en/ark/products/192446/intel-xeon-gold-6248-processor-27-5m-cache-2-50-ghz.html>`_,
while the production server (where the final submission lives) has `Intel Xeon E5-2650 <https://ark.intel.com/content/www/us/en/ark/products/64590/intel-xeon-processor-e52650-20m-cache-2-00-ghz-8-00-gts-intel-qpi.html>`_.