Sphinx guidelines
*****************

Introduction
############

Sphinx_ is a tool that makes it easy to create intelligent and beautiful
documentation, written by Georg Brandl and licensed under the BSD license.

The Sphinx_ is very popular documentation tool providing nice features
to create documentation especially for software projects. This documentation
is created with Sphinx_ as well as documentation for some other project
such as `WadersOS <https://waders-infra.gitlabe2-pages.ext.net.nokia.com/waders-docs/>`_
or `genapi <https://genapi.gitlabe1-pages.ext.net.nokia.com/>`_ and also
application teams are using it such as `BOAM <http://oam.emea.nsn-net.net/>`_.


Getting started
###############

reStructuredText
----------------

The Sphinx_ is using reStructuredText_ format. It is good to read through
basics and also have a cheat sheet at your hands at least at the beginning.
There are lots of quick guides and tutorials on the internet:

- `Step 1: Getting started with RST <https://sphinx-tutorial.readthedocs.io/step-1/>`_
- `reStructuredText User Documentation <https://docutils.sourceforge.io/rst.html>`_
- `reStructuredText (RST) Tutorial <https://www.devdungeon.com/content/restructuredtext-rst-tutorial-0>`_
- `rst-cheatsheet <https://github.com/ralsina/rst-cheatsheet/blob/master/rst-cheatsheet.rst>`_


Prerequisites
-------------

The Sphinx_ is implemented in Python therefore `Python` must be installed in your system.
If below sphinx-template_ is used then it is enough if `tox` is installed in your system.
Else you can also follow the Sphinx_ documentation how to setup your environment.


First documentation
-------------------

For convenience there is sphinx-template_ project which can be used
as starting point for new documentation or as a playground when learning
Sphinx_ or when trying out new plugins, themes etc.

To build and try your first Sphinx_ documentation run following.

::

    git clone git@gitlabe2.ext.net.nokia.com:rcp/templates/sphinx-template.git
    cd sphinx-template
    tox

This will generate the documentation out of the project which you can then
access locally with your browser. The URL to your local file system will
be printed on console.

Troubleshooting
###############

At start it might be tricky to understand the Sphinx_ error output if it
fails to parse your text. Usually it is good to do changes initially with
small increments and re-generate documentation frequently to know what
changes you have done and could possibly cause errors.  Also as Sphinx_
is widely used usually searching internet with the error text will provide
answers. Be prepared that especially at the beginning it might take bit
time to find out why your text formatting is broken.

Most parsing errors are caused by indentation errors or by using Chinese
characters.

.. _Sphinx: https://www.sphinx-doc.org
.. _reStructuredText: https://www.sphinx-doc.org/en/master/usage/restructuredtext/index.html
.. _sphinx-template: https://gitlabe2.ext.net.nokia.com/rcp/templates/sphinx-template
