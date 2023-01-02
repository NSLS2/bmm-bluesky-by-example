
.. |nd|      unicode:: U+2013  .. EN DASH
.. |nbsp| unicode:: 0xA0 
   :trim:


What is Redis?
==============

In this section, Redis will be explained in the specific context of
how it is used in day-to-day operations at BMM.  There is certainly
more to Redis than what is presented here, but this should be a useful
overview for any beamline scientist.

`Redis <https://redis.io/>`__ is a persistent, network-accessible data
store.  So ... what does that mean?

First off, Redis is a process that is running on one of BMM's IOC
servers.  Like an IOC, Redis is therefore accessible by any other
computer in the beamline LAN.  Specifically, Redis can be accessed by
any workstation, IOC server, VM, or display machine at the beamline.

The simplest way to use Redis (in fact, the way that Redis is used at
BMM), is to consider it as a way of storing keyword/value pairs.  Here
is an example using the Bash command line interface to Redis::

    redis-cli -h xf06bm-ioc2 SET mykey 2

Here's ``xf06bm-ioc2`` is the name of the server running Redis.  A key
called ``mykey`` is given the value of 2.  To retrieve this value::

    redis-cli -h xf06bm-ioc2 GET mykey

Entering that command will display ``2`` in the terminal.

That's about it.  A few comments:

#. As long as the server running Redis doesn't go away, the value of
   ``mykey`` will be retained.  Thus, that keyword/value pair will
   persist for a long time.  The server stores its own state, so
   ``mykey`` and its value will persist through server reboots.

#. Any computer on the LAN will have access to ``mykey``.  Thus
   different Bluesky processes on a single machine can see ``mykey``
   and it's value.  Bluesky processes on different machines can see
   ``mykey`` and it's value.  Processes that are not Bluesky can see
   ``mykey`` and it's value.

#. Any process on any machine can be monitoring the state of ``mykey``
   **and** update it.  When updated, all processes on all other
   machines will have access to ``mykey`` and its new value.



How Redis is used at BMM
------------------------

At BMM, Redis is currently used to handle three situations:

#. Maintaining the state of the current experiment when the Bluesky
   process is restarted after being stopped or after a crash.

#. Maintaining the state of an experiment that is being controlled by
   both an instance of Bluesky and an instance of `QueueServer
   <https://blueskyproject.io/bluesky-queueserver/>`__.

#. Providing information about the current experiment to an
   independent `beamline dashboard
   <https://nsls-ii-bmm.github.io/BeamlineManual/intro.html#ca-dashboard>`__
   running of several of the workstations at BMM.

Here are some examples of the types of information that are stored
with Redis at BMM.

Information pertaining to the user experiment

  This includes things like the PI's name, the proposal number, the
  SAF number, the start date of the experiment, the names of the
  people posted on the SAF, and the paths to the folders where data is
  being stored during the experiment.  Storing that type of
  information on Redis makes it much easier to recover from Bluesky
  crash or restart.

Aspects of the photon delivery system that cannot be ascertained simply by examining motor positions

  For example, the element and edge symbols of the most recent XAFS
  measurement are recorded.  Simply examining the position of the
  monochromator is not sufficient to identify the intended
  configuration of the beamline and the experiment.

  The contents of the sample holder containing energy calibration
  references is also stored on Redis.  On occasion, it is necessary to
  change the reference sample holder.  This is most commonly needed
  when measuring radiological materials containing elements not stored
  on the normal sample wheel.

Information identifying the type of experiment being run

  At BMM, a large fraction of the experiments fall into one of a few
  well-defined kinds of automation.  These tend to be related to
  specific sample holders or `in situ` instruments such as temperature
  stages.  Information about the type of automation and the current
  state (sample position, a string identifying the temperature stage,
  etc) is stored.  


Here is an example of one of the ways Redis is in daily use at BMM.
This is a screen shot of the beamline dashboard mentioned above.  This
is a simple python script using `PyEpics
<https://pyepics.github.io/pyepics/overview.html>`__ and Redis to
display the state of the beamline.

While most of the information being displayed is from direct querying
of EPICS PVs, a few things come from Redis, including element and edge
(in yellow text), the identification of the reference material (``Ref:``
|nbsp| ``Sr``) is from a combination of Redis and EPICS, and the knowledge to
display the wheel position (``wheel`` |nbsp| ``1``, a parameter in Redis
identifies the sample wheel as the sample holder currently in use).

.. _fig-redis-dashboard:
.. figure:: ../_static/dashboard.png
   :target: ../_static/dashboard.png
   :align: center

   The non-Bluesky, non-CSS heads-up display at BMM showing beamline
   state in a very compact way.  (This example was captured during a
   shutdown period |nd| there is no beam, shutters are closed, and the
   photon delivery system is in an odd state.  This picture should be
   replaced.) 

