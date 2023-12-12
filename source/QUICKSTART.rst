.. admonition:: Beta Release
   :class: error

   **Please be advised that our service is currently in its beta release of development.** As a beta user, your feedback and suggestions are highly valuable in helping us identify and address any issues. 
   We kindly request your patience and understanding as we work diligently to enhance the service based on your input.

Quickstar guide for NRP
=======================

This quickstart guide will walk you through the steps to get started with the NRP service.

You will learn how to:

- Create a project with the neurolibre `template repository <https://github.com/neurolibre/template>`_
- Using `RoboNeuro <https://robo.neurolibre.org/>`_ to test your project.

For the full process, please refer to the remainder of the author guide.

Prerequisites
-------------

.. sidebar:: Set up GitHub credentials

   If you do not have a GitHub account, please create one and configure your credentials by following the instructions `here <https://docs.github.com/en/github/getting-started-with-github/set-up-git>`_.


To go through this guide, you will need:

- A GitHub account and GitHub credentials set up on your local machine.
- Basic understanding of how to use ``git`` commands ``clone``, ``add``, ``commit``, ``push``.
- Basic Python skill.
- A Jupyter notebook.

Step by step guide
------------------

1. Create the a neurolibre compatible project from the template repository.
    Got to the neurolibre `template repository <https://github.com/neurolibre/template>`_ and click the :kbd:`use this template` button on the template repository page. 
    The template should have all the necessary files and folders to get you started, and a basic example of how to edit each file.

2. Clone the project on your local machine.
    Click on the :kbd:`code` button on your repository page and copy the link. Then run the following command in your terminal.

    .. code-block:: bash

        git clone <what you copy-pasted>
        cd <your repository name>

3. Update `_config.yml`.
    You will need to update at least the following fields:

    .. code-block:: yaml

        title                       : "NeuroLibre preprint template"  # Add your title
        author                      : John Doe, Jane Doe  # Add author names
        repository:
            url: https://github.com/neurolibre/template  # The URL to your new repository

.. admonition:: Minimal example

    Once these fields are updated, this is enought for an minimal test on `RoboNeuro <https://robo.neurolibre.org/>`_.
    Skip to step 9 and 10 to see how to test your project.


4. Add your executable content under ``content/``
    - ✅ You can add a mixture of ``Jupyter Notebooks``, ``MyST`` formatted markdown and plain text markdown files.
    - ✅ You can organize your content in subfolders.
    - ❌ We don't accept (non-executable) plain text markdown files alone.

    We recommand to start testing with a project without the need to download data for demonstrative purpose.
    Please follow the full documentation to learn how to add data to your project.

5. Edit ``_toc.yml`` to add your new executable content.
    You can follow the example in the template repository.

6. Update ``binder/requirements.txt`` to add your new dependencies.
    Make sure you keep the ``Jupyter Notebook`` and ``Repo2Data`` dependency.
    Add other dependencies based on your executable content.

7. Edit ``paper.md`` and ``paper.bib`` to add your new content.
    You can follow the example in the template repository.

8. Build the project locally to test your changes. [Optional]
    .. code-block:: bash

        jupyter-book build content
    You can now open the ``content/_build/html/index.html`` file in your browser to see the result.

9. Commit and push the changes to your repository.
    .. code-block:: bash

        git add .
        git commit -m "update author information"

10. Go to `RoboNeuro <https://robo.neurolibre.org/>`_ and click on the :kbd:`Complie preprint` button.
    From here you can iterate on your project by going back to step 4!

