.. warning:: NeuroLibre is at an alpha stage of development, and is not currently open for submissions.

Step 2 - 🖥️ Test your submission
================================

Local testing is checking notebooks are running, binder and jupyter book is building.
First locally, then second directly on neurolibre servers.

------------

Local testing
:::::::::::::

It is really important to first test your submission locally to alleviate further issues when deploying on Neurolibre server.
You need to make sure that:

* All the notebooks run locally with the hardware requirements from computation and data section.
* The jupyter book builds fine locally (make sure that you are not using cache files).
* The interactive environment build correctly on https://mybinder.org/.

To help you testing your book locally, you can follow the section on :doc:`LOCAL_TESTING`.

Testing on NeuroLibre servers
:::::::::::::::::::::::::::::

Meet ``RoboNeuro``! Our preprint submission bot is at your service 24/7 to help you create a NeuroLibre preprint.

.. image:: https://github.com/neurolibre/brand/blob/main/png/preview_magn.png?raw=true
  :width: 500
  :align: center

We would like to ensure that all the submissions we receive meet certain requirements. To that end, we created the `RoboNeuro preview service <https://roboneuro.herokuapp.com>`_, 
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
Our binder submission page is available here: https://binder.conp.cloud.

When this process is really usefull for debugging your submission live, it can be verry long to get it.
Indeed, a jupyter book build will always occur under the hood, and as part of the build process it will try to execute everything within your submission.
This can make the build process very long (especially if you have a lot of long-running notebooks), and so you will end up waiting forever to get the binder instance.

If you are in a case where the jupyter book build fails on Neurolibre for whatever reason but works locally,
you can bypass the jupyter book build to get the interactive session almost instantly.

.. note:: For example if you have “out of memory” errors on Neurolibre, you can reduce the RAM requirements on the interactive session, and try to re-run the jupyter book build directly on the fly.

Just add ``--neurolibre-debug`` in your latest commit message to bypass the jupyter book build (as in `this git commit <https://github.com/ltetrel/nimare-paper/commit/4d5938819ad0a21365bc849ab91d29211556c77d>`_).
Now if you register your repository on https://binder.conp.cloud, you will have your binder instance almost instantly.
You should be able to open a terminal session or play with the notebooks from there.

.. note:: This setup requires a previous valid binder build. If you are not able to build your binder, then you don't have a choice to fix the installation locally on your PC.

.. warning:: Please remember to remove the flag ``--neurolibre-debug`` when you are ready to submit, since NeuroLibre needs to build the jupyter book.