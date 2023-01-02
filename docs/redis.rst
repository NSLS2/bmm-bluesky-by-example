
.. |nd|      unicode:: U+2013  .. EN DASH

Using Redis to store beamline state persistently
================================================

.. admonition:: Section Topic

   This documents one way of using `Redis <https://redis.io/>`__ at a
   beamline.  Redis is an in-memory data store which can be served
   from a server at the beamline.  At BMM, Redis is used to store
   keyword/value pairs |nd| a dict, in essence |nd| as a way of
   maintaining the state of an experiment.

   :Lesson: Maintain the state of an experiment in a way that is
	    independent of the running Bluesky process.  In this way,
	    state can be preserved even if Bluesky crashes of if
	    multiple processes require knowledge of the state of the
	    running experiment.

   :Beamline: `BMM <https://wiki-nsls2.bnl.gov/beamline6BM/index.php?Main_Page>`__

   :Link to source code: `linkam.py <https://github.com/NSLS-II-BMM/profile_collection/blob/master/startup/BMM/workspace.py>`__

   :Section author: Bruce Ravel (bravel@bnl.gov)


.. toctree::
   :maxdepth: 1
   :caption: Contents:

   redis/whatis.rst
   redis/state.rst
   redis/codelisting.rst

