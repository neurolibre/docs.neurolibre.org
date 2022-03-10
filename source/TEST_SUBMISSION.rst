.. warning:: NeuroLibre is at an alpha stage of development, and is not currently open for submissions.

2. Test your submission
=======================

Local testing is checking notebooks are running, binder and jupyter book is building.
First locally, then second directly on neurolibre servers.

Local testing
:::::::::::::

It is really important to first test your submission locally to alleviate further issues when deploying on Neurolibre server. You need to make sure that:
All the notebooks run locally with the hardware requirements from computation and data req section.
The jupyter book builds fine locally (make sure that you are not using cache files).
The interactive environment build correctly on https://mybinder.org/.

To help you testing your book locally, you can follow this section https://docs.neurolibre.org/en/latest/SUBMIT.html#testing-book-build-locally.

Testing on NeuroLibre servers
:::::::::::::::::::::::::::::

Meet ``RoboNeuro``! Our preprint submission bot is at your service 24/7 to help you create a NeuroLibre preprint.

.. image:: https://github.com/neurolibre/brand/blob/main/png/preview_magn.png?raw=true
  :width: 500
  :align: center

We would like to ensure that all the submissions we receive meet certain requirements. To that end, we created a `test page <https://roboneuro.herokuapp.com>`_, 
where you point RoboNeuro to a public GitHub repository, enter your email address, then sit back and wait for the results.

.. note:: RoboNeuro book build process has two stages. First, **it creates a virtual environment** based on your runtime descriptions. If this stage is successful, then it proceeds to 
          build a Jupyter Book by **re-executing your code** in that environment. 

- On a successful book build, we will return you a preprint that is beyond PDF!
- If the build fails, we will send two log files for your inspection.

.. warning:: **A successful book build on RoboNeuro preview service is a prerequisite for submission.**

            Please note that RoboNeuro book preview is provided as a public service with limited computational resources. Therefore we encourage you to build your book locally before
            requesting our online service. Instructions are available `here <#testing-book-build-locally>`_.

Debugging for long NeuroLibre submission
::::::::::::::::::::::::::::::::::::::::

....