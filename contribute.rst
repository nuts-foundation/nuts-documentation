##########
Contribute
##########

If you want to contribute to any of the nuts foundation projects or to this documentation, please fork the correct project from `Github <https://github.com/nuts-foundation>`_ and create a pull-request.

***************************
Documentation contributions
***************************

Documentation is written in Restructured Text. A CheatSheet can be found `here <https://thomas-cokelaer.info/tutorials/sphinx/rest_syntax.html>`_.

You can test your documentation by installing the required components. You first have to install python, check `<https://www.python.org/>`_ on how to install python for your OS.
Next you have to install the following components using **pip**::

    pip install sphinx --user
    pip install recommonmark
    pip install sphinx_rtd_theme

Then you can generate the documentation locally with::

    make html

For small changes you might want to add the *clean* directive::

    make clean html

The documentation will then be available from ``_build/html/index.html``
