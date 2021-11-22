
Introduction
============

Running a beamline is hard!  There are so many things that have to be
working correctly for a beamline to work correctly.  One of them is
the data acquisition software.  Happily, we here at NSLS-II get to use
Bluesky and it's companion libraries Ophyd and Databroker.  These are
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
https://github.com/bruceravel/bluesky-book, write up your own content,
and submit a pull request to have your content included.  The more
voices heard, from the better.

Submit short examples or lengthy deep dives.  Show how you turned
busy work into something simple and routine.  Show how to interface to
a new piece of scientific instrumentation.  Show how to incorporate
machine learning into the Bluesky workflow.  Show how you display
information to your users in a convenient way.  No examples are too
big or too small.

To get started, first install `sphinx <http://www.sphinx-doc.org/>`__
and the {book}theme `as explained here
<https://sphinx-book-theme.readthedocs.io/en/latest/tutorials/get-started.html>`__

Once you have downloaded a fork of `the repository
<https://github.com/bruceravel/bluesky-book>`__, edit away.  You can
check that your contributions build correctly by going to the ``docs``
folder and running ``make html``.  Once your contributions contain all
your wisdom and build correctly, go ahead and make a pull request.


Style guide
-----------

If you contribute, please follow the existing examples:

#. Make a sub-folder to contain the rst files of your chapter.

#. Put a index file for your chapter in the top directory.  That index
   file should have the same name as your sub-folder.

#. Follow the content of the admonition block at the top of the index
   file so the reader can understand the intent of the chapter, find
   the original source code, and appreciate you, the brilliant,
   wonderful person sharing your knowledge.

#. If your contribution is quite short and can be contained in a
   single rst file, simply put that rst file in the top folder without
   creating a sub-folder for your content.  If, however, your content
   can reasonably be split up among various files and placed in a
   sub-folder, it is better to do that.


Notes
-----

This documentation project uses the lovely {book}theme from
https://sphinx-book-theme.readthedocs.io/en/latest/index.html
and `The Executable Book Project <https://ebp.jupyterbook.org/>`__.
