Nuts Documentation
==================

This project contains the public documentation for Nuts.

> :warning: **DEPRECATED** This documentation is for the first Nuts node with the Corda softare. The new version can be found here: https://github.com/nuts-foundation/nuts-node/. 


To generate the documentation locally follow the following steps:

.. inclusion-marker-for-contribution

You first have to install python, check `<https://www.python.org/>`_ on how to install python for your OS.

.. note::

    MacOS (currently) comes with Python 2 preinstalled which has been deprecated as of January 1st, 2020.
    It's recommended to upgrade to Python 3 before proceeding.

Next you have to install `pip <https://pip.pypa.io/en/stable/installing/>`_.
Then install the following components using **pip**::

    pip install sphinx --user
    pip install recommonmark
    pip install sphinx_rtd_theme
    pip install sphinxcontrib.httpdomain
    pip install sphinx-jsonschema
    pip install rst_include

For MacOS make sure the sphinx executables are added to your PATH (e.g. for Python 3.8):

    export PATH=$HOME/Library/Python/3.8/bin:$PATH

Then you can generate the documentation locally with::

    make html

For small changes you might want to add the *clean* directive::

    make clean html

The documentation will then be available from ``_build/html/index.html``
