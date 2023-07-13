.. admonition:: Beta Release

   **Please be advised that our service is currently in its beta release of development.** As a beta user, your feedback and suggestions are highly valuable in helping us identify and address any issues. 
   We kindly request your patience and understanding as we work diligently to enhance the service based on your input.

NeuroLibre Reproducible Preprints
==========

.. image:: https://github.com/neurolibre/brand/blob/main/png/card_tb.png?raw=true
     :align: left
     :width: 400

As a registered preprint publisher, NeuroLibre goes beyond the traditional boundaries of research dissemination by offering
NeuroLibre Reproducible Preprints (NRPs).


.. rubric:: Code, data, and computational runtime are not supplementary but rather integral components of published research.

Embracing this principle, NRPs are built by seamlessly combining the outputs of your preprint's executable content with the scientific prose, 
all within the same execution runtime required for your analyses.

|
.. rubric:: Moving from static PDFs with code and data availability statements to NRPs is the giant leap that modern research yearns for. With NeuroLibre, we've cracked the code to make that leap a walk in the park.


.. admonition:: Explore a published NRP

   https://doi.org/10.55458/neurolibre.00004

Bird's eye view of NRP publication workflow
-----------------

.. sidebar:: 🚀 NRPs are dynamic

   Enabling you not only to execute NRPs directly in your web browser and regenerate the figures from the preprint,
   but also providing support for interactive figures and even dashboards!
   
   This interactivity allows for a more immersive exploration of the preprint, 
   whether by reviewers upon peer-reviewed journal submission, or by researchers who are interested in your work.

To **submit** an NRP you need to provide the following:

1. A **public code repository** that has a single or a collection of Jupyter Notebooks and/or `MyST markdown <https://mystmd.org>`_ files.
2. A **public data repository** needed to generate the outputs (typically figures) from the executable part of your content.
3. **Reproducible runtime configurations** recognized by `BinderHub <https://mybinder.readthedocs.io/en/latest/using/config_files.html>`_. 
4. A bibtex formatted **bibliography** (paper.bib) and **author information** (paper.md).

Using your ORCID, you can login to NeuroLibre's submission portal and fill out a simple form. After **content moderation**, 
NeuroLibre starts a **technical screening process**, which takes place on GitHub using NeuroLibre's one-of-a-kind editorial
workflow, powered by the OpenJournals.

.. sidebar:: ♻️ NRPs are FAIR

   NeuroLibre archives all the necessary reproducibility assets required to successfully build an NRP at the time of submission.

   NeuroLibre's production BinderHub, container registry, data storage, and web servers are reserved for the published NRPs, ensuring
   the long-term functionality and accessibility. This makes NRPs Findable, Accessible, Interoperable, and Reusable.


During the technical screening process, our editorial bot RoboNeuro and a screener works with you to ensure a successful build 
of your NRP on NeuroLibre test servers.

After a successful build, the following reproducibility assets are transferred from our preview server (public) to our production servers 
(reserved for published NPRs only) and archived individually on Zenodo:

1. Docker image
2. Dataset (unless already archived)
3. Repository (version cut at the latest successful build)
4. Built NRP (HTML pages of the executable book)

Each of these reproducibility assets is attributed to every author on the NRP, and they are assigned a DOI (Digital Object Identifier).

.. sidebar:: ➕ More than another Binder/Jupyter instance

   In addition to providing a seamless publication workflow for reproducible preprints and officially registering them for discoverability
   of scholarly content, NeuroLibre overcomes an important reproducibility roadblock. For example, without the bundling of reproducibility 
   assets within a dedicated publication framework:

   * Public container registries wipe out images.
   * Future attempts to build the same images (like Binder) frequently stumble over version clashes, resulting in failures.
   * Unless cached on the same server where JupyterHub runs and archived, data often slips into the realm of inaccessibility.

   As a next-generation publisher, NeuroLibre ensures that your preprint retains its reproducibility prowess. With our robust framework, 
   we preserve and safeguard all necessary assets, leaving no room for disappearing images, version woes, or elusive data. Rest easy, 
   knowing your preprint remains reproducible and readily available to all.

Once the archival process is complete, a summary PDF is generated. This PDF is necessary to officially register NPRs as preprints. All the 
archived reproducibility assets, cited references, and the link to the reproducible preprint are resource linked to the DOI assigned by NeuroLibre 
upon publication (DOI: 10.55458/neurolibre).

Similar to that in traditional preprint repositories (e.g. arXiv), NeuroLibre updates metadata relationship to an Author Accepted Manuscript (AAM) or
Version of Record (VoR) after your article has been accepted for publication by a journal, following the peer review process and any revisions 
requested by the reviewers or editors.


For authors, it takes three steps: ``prepare``, ``test`` and ``submit``
:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
Our full-fledged (yet simple) submission workflow is designed to help researchers convert a collection of 
Jupyter Notebooks and Markdown files into publishable content. 

Providing online-executable runtimes and a powerful data caching system, NeuroLibre makes a **groundbreaking preprint
server where you can plant the seeds of living publications.**

As an author you should follow:

* :doc:`PREPARE`
* :doc:`TEST_SUBMISSION`
* :doc:`SUBMIT`

For readers, it takes one click!
::::::::::::::::::::::::::::::::
The interactive outputs (such as interactive figures) are readily available in NeuroLibre preprints. Yet, if you'd like
to re-run the analyses and reproduce the outputs, our powerful cloud infrastructure is at the tip of your fingers.

.. seealso:: To find out more about how to navigate a NeuroLibre preprint, you can visit :doc:`READER`.

Contributions are welcome!
::::::::::::::::::::::::::
NeuroLibre is fully open-source and draws its strength from community-developed tools such as `BinderHub <https://github.com/jupyterhub/binderhub>`_ and `Open Journals <https://github.com/openjournals>`_.
You can find more information under our `github organization <https://github.com/neurolibre>`_.

.. seealso:: If you are interested in contributing to NeuroLibre or to deploy one of your own, please visit `developer documentation <BAREMETAL_TO_BINDERHUB>`_.

.. image:: https://github.com/neurolibre/brand/blob/main/png/groundbreaking.png?raw=true
  :width: 500
  :align: center
  :alt: NeuroLibre preprints are beyond PDFs 

