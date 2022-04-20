Submission workflow backend
===========================

The submission workflow backend has several components that are still not part of the terraform installation.
It is divided into multiple parts:
* the data server which serves the jupyter books
* the python API to communicate with the bind, communicate with the API, archive and deal with DOIs.
* front-end for the website design, forked from `JOSS <https://joss.theoj.org/>`_

Data server and python API
::::::::::::::::::::::::::

You will find all instructions in the github repo: https://github.com/neurolibre/neurolibre-data-api
It has two branches: ``main`` for the test server and ``prod`` for the production server.