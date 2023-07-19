.. admonition:: Beta Release
   :class: error

   **Please be advised that our service is currently in its beta release of development.** As a beta user, your feedback and suggestions are highly valuable in helping us identify and address any issues. 
   We kindly request your patience and understanding as we work diligently to enhance the service based on your input.

Submit your NRP to NeuroLibre
==================

.. admonition:: Before you submit
   :class: hint
   
   Before submitting your NRP, please make sure that your GitHub repository adheres to the :doc:`STRUCTURE`.
   It is ``RECOMMENDED`` for the authors to test the functionality of their NRPs locally, then using the RoboNeuro test service (https://robo.neurolibre.org).

To submit your NRP:

- Login to the submission portal on https://neurolibre.org by using your **ORCID ID** (``REQUIRED``).
- Click the submit button (the one on the top bar, or the fancier one on the banner).

Submission form includes the following fields:

* :kbd:`Title` Please provide the same title provided in your ``content/_config.yml``.
* :kbd:`Repository` Please provide the GitHub URL to your NRP repository.
* :kbd:`Branch` We recommend leaving this field empty, which defaults to the ``main`` branch of your NRP repository, which is expected to be the most up to date before submission.
* :kbd:`Software version`: If you have a release tag corresponding to the version of your submission, please indicate. Put ``v1.0`` otherwise.
* :kbd:`Main subject of the paper`: Select ``mri``, ``fmri`` (the list will be extended).
* :kbd:`Type of the submission`: Select ``New submission``
* :kbd:`Preprint data DOI`:  Please provide if your dataset has already been given a DOI.
* :kbd:`Message to the editors`: Briefly inform our content moderators about your submission in a few sentences, please keep it short.

After your submission, the managing editor will initiate a pre-review issue in the `NeuroLibre reviews repository <https://github.com/neurolibre/neurolibre-reviews>`_, provided that the 
content moderation is successful. During the pre-review, a **technical screener** will be assigned to your submission and the "review" will be started, which
is a GitHub issue on the reviews repository.

.. admonition:: Technical screening vs peer review
   :class: warning

   Technical screening is conducted to verify the functionality of your NRP. 
   As a preprint publisher, NeuroLibre does not assess the scientific content of the preprint.