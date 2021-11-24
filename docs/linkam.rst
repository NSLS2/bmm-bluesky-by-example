
An Ophyd device for the Linkam T96
==================================

.. admonition:: Section Topic

   This documents the Ophyd class for the `Linkam
   <https://www.linkam.co.uk/temperature-controlled-stages>`__ T96
   temperature stage controller developed for use at BMM.

   :Lesson: Turn a controller and a sensor into an Ophyd
	    positioner. This allows you to do something like this at
	    the command line:

	    .. code-block:: python

	       RE(mv([linkam], 250))

	    or this in a Bluesky plan:

	    .. code-block:: python

	       yield from mv([linkam], 250)

	    This example also explains how to compute a custom signal
	    indicating that the positioner is done moving.

   :Beamline: `BMM <https://wiki-nsls2.bnl.gov/beamline6BM/index.php?Main_Page>`__

   :Link to source code: `linkam.py <https://github.com/NSLS-II-BMM/profile_collection/blob/master/startup/BMM/linkam.py>`__

   :Section author: Bruce Ravel (bravel@bnl.gov)


.. toctree::
   :maxdepth: 1
   :caption: Contents:

   linkam/problem.rst
   linkam/ophyd.rst
   linkam/features.rst
   linkam/codelisting.rst

