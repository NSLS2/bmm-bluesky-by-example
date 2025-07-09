
Introduction
============

Running a beamline is hard!  There are so many things that have to be
working correctly for a beamline to work correctly.  One of them is
the data acquisition software.  Happily, we here at NSLS-II get to use
Bluesky and its companion libraries Ophyd and Databroker.  These are
great tools -- robust, well supported, actively developed, extensively
documented. 

While the documentation is thorough and provides good coverage of the
code, it is software documentation.  It documents the technical
implementation of the various bits and bobs that make up the Bluesky
ecosystem. 

This is an effort to gather together a different kind of
documentation.  This catalogs efforts by beamline scientists to solve
problems at the beamline.  This is a how-to guide and a
lessons-learned guide.  This contains extensive code examples with
commentary on the intent of the code and explanations of why things
work the way they work.  This is an effort to show beamline scientists
how to solve the sorts of problems encountered daily at the beamline.

Ideally, this is a font of solutions that beamline staff can implement
immediately and use as inspiration for solving other beamline
problems.

Contributing to this effort
---------------------------

Please feel free to fork this from
https://github.com/NSLS-II-BMM/bluesky-by-example, write up your own
content, and submit a pull request to have your content included.  The
more voices heard from, the better.

Submit short examples or lengthy deep dives.  Show how you turned
busy work into something simple and routine.  Show how to interface to
a new piece of scientific instrumentation.  Show how to incorporate
machine learning into the Bluesky workflow.  Show how you display
information to your users in a convenient way.  No examples are too
big or too small.

To get started, first install `sphinx <http://www.sphinx-doc.org/>`__
and the {book}theme `as explained here
<https://sphinx-book-theme.readthedocs.io/en/latest/tutorials/get-started.html>`__

Next install these Sphinx extensions:

* ``sphinx-math-dollar`` to render equations and mathematical symbols

..
  * ``sphinxemoji`` to render cute emojis

by doing this:

.. code-block:: bash

   pip install sphinx-math-dollar

..
   pip install sphinxemoji
   Alas, Github actions was unable to install this package.  Sigh...

Download a fork of `the repository
<https://github.com/NSLS-II-BMM/bluesky-by-example>`__ and edit away!
You can check that your contributions build correctly by going to the
``docs`` folder and running ``make html``.  Once your contributions
contain all your wisdom and build correctly, go ahead and make a pull
request.


Style guide
-----------

If you contribute, please proofread and please follow these style
guidelines:

#. Choose an evocative name for your contribution.  That name will be
   used for your contribution's index file and for the subfolder
   containing your content.

#. Copy the ``NewThing`` sub-folder to a sub-folder with your
   evocative name.  Note that the ``NewThing`` folder comes with some
   example ``.rst`` files.  You do not need to structure your
   contribution in this way -- do whatever best tells your story.  Be
   sure that the table of contents in your index file points to each
   of the files in your sub-folder.

#. Copy the ``NewThing.rst.example`` index file to an ``.rst`` file
   with your evocative name in the top directory.  Note that the index
   file should have the same name as the sub-folder.

#. Fill in the content of the admonition block at the top of the index
   file so the reader can understand the intent of the chapter, find
   the original source code, and appreciate you, the brilliant,
   wonderful person sharing your knowledge.

#. If you label sections, figures, tables, etc in your contribution
   using `Sphinx cross-referencing
   <https://docs.readthedocs.io/en/stable/guides/cross-referencing-with-sphinx.html>`__,
   you should use the name of your folder/index file in the
   reference label to avoid label name collisions.  For example, in
   the section on the Ophyd interface for the Linkam stage, all the
   labels contain the string "linkam".

#. If your contribution is quite short and can be contained in a
   single rst file, simply put that rst file in the top folder without
   creating a sub-folder for your content.  Follow the example of
   ``colored_text.rst``.  Note that the one-pager should still have
   the admonition block at the beginning to explain intent and
   authorship.  If, however, your content can reasonably be split up
   among various files and placed in a sub-folder, it is better to do
   that.

Beyond that, feel free to include material on any topic, using any
feature of the Sphinx system.  In particular, note that the
`{book}theme
<https://sphinx-book-theme.readthedocs.io/en/latest/index.html>`__
makes special provision for `Jupyter content
<https://sphinx-book-theme.readthedocs.io/en/latest/notebooks.html>`__.

Notes
-----

This documentation project uses the lovely `{book}theme
<https://sphinx-book-theme.readthedocs.io/en/latest/index.html>`__
from the `The Executable Book Project
<https://ebp.jupyterbook.org/>`__.
