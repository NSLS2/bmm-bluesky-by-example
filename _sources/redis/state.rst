.. |nd|      unicode:: U+2013  .. EN DASH

Store and retrieve state information
====================================

To start, you must have Redis running on a machine on the LAN.  This
is pretty simple on the NSLS-II machines as Redis is an available
package.  A DSSI staff member can get it installed such that the REDIS
server autostarts when the computer boots.

There is a python package for connecting to a Redis server that is a
standard part of the beamline Conda environments.  Here's the python
code to drop into a Bluesky profile:

.. code-block:: python
   :linenos:

     import redis
     rkvs = redis.Redis(host='xf06bm-ioc2', port=6379, db=0)

Here, ``xf06bm-ioc2`` is the DNS-resolvable name of the server and
``6379`` is the default port number that the Redis server runs on.
``rkvs`` is an acronym for "Redis keyword/value store".

Once armed with a connection to the host

.. code-block:: python

     rkvs.set('BMM:pds:element', 'Cu')

and 

.. code-block:: python

     el = rkvs.get('BMM:pds:element')

In this example, a parameter of the state of the photon delivery
ssytem (``pds``) is stored to identify the element currently being
measured by XAFS.

A pitfall to using the python interface to redis is that strings are
always returned as `byte strings
<https://docs.python.org/3/library/stdtypes.html#bytes-objects>`__,
which allows for handling unicode text.  To convert the byte string to
a normal string, do:

.. code-block:: python

     el = rkvs.get('BMM:pds:element').decode('utf-8'))


At BMM, the adopted convention is to structure state data in a manner
similar to a dictionary of dictionaries, but with the keywords
flattened.  Thus, the keyword ``BMM:pds:element`` is interpreted as:

#. A parameter related to the BMM beamline.  This leaves open the
   possibility of using the Redis server to store information about
   other things |nd| other beamlines or things that are not
   beamlines.  For example, in a recent series of experiments in
   collaboration with the PDF beamline, state information is shared
   between the beamlines.  In that case, we might store keywords like
   ``PDF:something:something`` in BMM's Redis store.

#. This parameter is related to the photon delivery system, hence the
   ``pds`` part of the key string.  Other strings used at BMM in this
   position are ``user``, ``lmfit``, and ``scan``.

#. The last portion of the keyword identifies the specific parameter.

Anytime something changes the state of the beamline, care is taken in
BMM's profile to also update the related value in Redis.

Thus, using Redis effectively in the Bluesky profile requires a
certain amount of discipline to keep the state of the Redis store
consistently up to date.


