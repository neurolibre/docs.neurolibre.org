.. admonition:: Beta Release
   :class: error

   **Please be advised that our service is currently in its beta release of development.** As a beta user, your feedback and suggestions are highly valuable in helping us identify and address any issues. 
   We kindly request your patience and understanding as we work diligently to enhance the service based on your input.

Test your NRP
================================

Prior to deployment on the Neurolibre server, it is crucial to thoroughly test your submission locally 
to prevent potential issues. Here are the essential steps to ensure a smooth deployment:

* Verify Local Notebook Execution:

Test all the notebooks locally, ensuring they run seamlessly with the specified hardware requirements 
mentioned in the computation and data section.

* Validate Jupyter Book Build:

Locally build the Jupyter book without relying on cache files to ensure it compiles successfully.
By meticulously testing your submission locally, you can preemptively address any problems and guarantee a 
reliable deployment experience on the Neurolibre server. This proactive approach will save time and ensure a 
smoother integration into the platform.

* Test your NRP on RoboNeuro

We provide a web service to test your NRP builds using the same infrastructure involved in the technical 
screening process.

Quick guide for local testing
:::::::::::::::::::::::::::::

With the necessary dependencies already installed for local notebook development and 
your preprint repository structured according to the :doc:`STRUCTURE`, conducting a local test of 
your preprint build becomes a straightforward process.

.. admonition:: Install Jupyter Book (prefer ``<= 0.14.0``)

   .. code-block:: shell

      pip install jupyter-book

.. admonition:: Describe your data dependencies 

   Given the following minimalistic repository structure:

   .. code-block:: shell

    ├── binder
    │   ├── requirements.txt
    │   └── data_requirement.json
    ├── content
    │   ├── _build
    │   ├── notebook.ipynb
    │   ├── _config.ym
    │   └── _toc.yml
    └── README.md

   Create a folder named ``data`` at the root of the repository and don't forget to add it to the .gitignore file to exclude it from version control:

   .. code-block

    /data 

   Install `Repo2Data <https://github.com/SIMEXP/Repo2Data>`_:

   .. code-block:: shell

    pip install repo2data

   Configure the ``dst`` from the requirement file (``binder/data_requirement.json``), so it points to the ``data`` folder you created:

   .. code-block:: json

    {"src" : "/url/to/the/online/source",
     "dst" : "../data",
     "projectName": "my_dataset"}

   Run ``repo2data`` inside your notebook and get the path to the data.

   .. code-block:: python
    import os
    from repo2data.repo2data import Repo2Data
    # Tell repo2data where to find the data_requirement.json
    data_req_path = os.path.join("..", "binder", "data_requirement.json")
    # Instantiate a repo2data object
    repo2data = Repo2Data(data_req_path)
    # Download the dataset and store data_path in a variable
    # Once the data is downloaded, successive runs will not re-attempt downloading it.
    data_path = repo2data.install()[0]

.. admonition:: Build your Jupyter Book

   .. code-block:: shell

    cd /your/repo/directory
    jupyter-book build ./content

   Please visit reference `documentation <https://jupyterbook.org/content/execute.html?highlight=execute#execute-and-cache-your-pages>`_ on executing and caching your outputs during a book build.

.. admonition:: See also: GitHub Actions to test book builds
   :class: hint
   
   You can take a look at `this example GitHub Actions file <https://github.com/rrsg2020/paper/blob/main/.github/workflows/main.yml>`_ to see how you can trigger
   a book build whenever a commit is pushed to the main branch of your NRP repository.

   Please be aware that the GitHub Actions workflow's necessary steps may vary depending on your specific requirements. It's essential to adapt the workflow accordingly to 
   suit your NRP's needs.

Testing on NeuroLibre servers
:::::::::::::::::::::::::::::

We are thrilled to present 🤖 RoboNeuro 🤖, our dedicated preprint submission bot, designed to assist you in effortlessly 
creating NeuroLibre preprints anytime you need. With RoboNeuro at your service round the clock, preprint submission becomes a breeze!

.. image:: https://github.com/neurolibre/brand/blob/main/png/preview_magn.png?raw=true
  :width: 500
  :align: center

This service allows you to evaluate whether your submission meets specific requirements before formal submission. 
It helps ensure that all contributions align with our guidelines and standards.

Using the RoboNeuro preview service is a straightforward process:

1. Access the service at https://robo.neurolibre.org.
2. Provide the URL of your public GitHub repository.
3. Enter your email address and the GitHub commit SHA for the build.
4. Wait for the service to process your request and send you the results via email.

By leveraging the RoboNeuro preview service, we aim to enhance the quality and consistency of submitted preprints.
It serves as a valuable tool for authors, allowing them to verify that their work aligns with the necessary criteria before 
proceeding with the formal submission process.

.. note:: The RoboNeuro book build process comprises two distinct stages:

   - Firstly, it establishes a virtual environment based on your specified runtime descriptions (``runtime build``). 
   - Upon successful completion of this stage, it proceeds to the second phase, where it builds the Jupyter Book by re-executing your code within that environment (``book build``).

🌱 **On a successful book build**, we will return you a https://preview.neurolibre.org URL that serves your NRP at the provided commit SHA from which it was built.

🥀 If the build fails, we will send you an html file that contains the following logs:
  - Binder build logs
  - Book build logs 
  - Execution logs of individual notebooks

.. admonition:: A successful book build on RoboNeuro preview service makes technical screening easier.
   :class: error

   Please note that RoboNeuro book preview is provided as a public service with limited computational resources. 
   Therefore we encourage you to build your book locally before using https://robo.neurolibre.org.

Debugging a long-running ``book build``
----------------------------------------

We offer https://test.conp.cloud, a Binder build page, where you can initiate a ``runtime build.``
By default, if the ``runtime build`` succeeds, it automatically proceeds to perform a ``book build.``

However, if the execution of all your notebooks take considerably long time (say more than 20 minutes), you may
want to bypass the ``book build`` phase to debug a particular notebook in the runtime environment built on our test
BinderHub. This is particularly useful in cases where the Jupyter book build fails on Neurolibre but works locally.

To bypass the ``book build``, simply include ``--neurolibre-debug`` in your latest commit message, as demonstrated in `this 
git commit <https://github.com/ltetrel/nimare-paper/commit/4d5938819ad0a21365bc849ab91d29211556c77d>`_. Upon a successful ``runtime build``, 
you can interact with your NeuroLibre Research Preprint (NRP) through the Jupyter Notebook interface in your web browser. This interface allows you to execute code cells 
in specific notebooks or run commands in the terminal as needed.

.. note:: If you run into ``out of memory`` errors on Neurolibre, you can reduce the RAM requirements on the interactive session, and try to re-run the jupyter book build in the same session.

.. admonition:: Do not forget to remove the debug flag
   :class: error 
   
   Once you have finished debugging your executable content on https://test.conp.cloud, make sure that your latest commit does not have 
   the ``--neurolibre-debug`` in its commit message. Otherwise, following requests will not trigger a ``book build``.