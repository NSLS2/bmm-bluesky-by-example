
.. _redis_codelisting:

Code Listing
============

Here is the full listing for a Redis connection in a Bluesky profile.

The definition of the redis interface `can be found here
<https://github.com/NSLS-II-BMM/profile_collection/blob/master/startup/BMM/workspace.py>`__
at BMM's GitHub site.

.. code-block:: python
   :linenos:

     import redis

     if not os.environ.get('AZURE_TESTING'):
        redis_host = 'xf06bm-ioc2'
     else:
        redis_host = '127.0.0.1'


     class NoRedis():
        def set(self, thing, otherthing):
           return None
        def get(self, thing):
           return None

     rkvs = redis.Redis(host=redis_host, port=6379, db=0)
     #rkvs = NoRedis()
     

Explanation
-----------

The block at lines 3-6 is a bit of care taken to make sure that
automated testing does not trigger false negatives due to being unable
to reach the Redis server at the beamline.  The sort of automated
testing in question tends to happen on servers located away from the
beamline. Redirecting the Redis connection to the local host, prevents
this false negative.

The class defined at lines 9-13 is a way to trick the profile into
starting and running on a machine that does not have access to the
Redis server.  in that case, uncomment line 16.  Not everything will
work correctly in this case, but the Bluesky session will at least
start without complaint.

Setting and retrieving a scalar value
-------------------------------------

This sets a scalar value:

.. code-block:: python

     rkvs.set('BMM:pds:element', 'Cu')

This retrieves a string and converts it to an ordinary string:

.. code-block:: python

     el = rkvs.get('BMM:pds:element').decode('utf-8'))


Setting and retrieving a list
-----------------------------

This sets a list value:

.. code-block:: python

     rkvs.set('BMM:example:list', ('a', 'b', 'c'))


This retrieves a list ...

.. code-block:: python

     mylist = rkvs.get('BMM:example:list')

(Is this quite right?)


Setting and retrieving a dictionary
-----------------------------------

This sets a list value:

.. code-block:: python

     rkvs.set('BMM:example:dict', ('a': 1, 'b': 2, 'c':3))


This retrieves a list ...

.. code-block:: python

     mylist = rkvs.get('BMM:example:dict')

(Is this quite right?)

