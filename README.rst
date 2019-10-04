Nuts Documentation
==================

This project contains the public documentation for Nuts.

To generate the documentation locally follow the following steps:

.. inclusion-marker-for-contribution

You first have to install python, check `<https://www.python.org/>`_ on how to install python for your OS.
Next you have to install `pip <https://pip.pypa.io/en/stable/installing/>`_.
Then install the following components using **pip**::

    pip install sphinx --user
    pip install recommonmark
    pip install sphinx_rtd_theme
    pip install sphinxcontrib.httpdomain

For MacOS make sure the sphinx executables are added to your PATH::

    export PATH=$HOME/Library/Python/2.7/bin:$PATH

Then you can generate the documentation locally with::

    make html

For small changes you might want to add the *clean* directive::

    make clean html

The documentation will then be available from ``_build/html/index.html``
