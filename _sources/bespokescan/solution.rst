
Making a bespoke measurement plan
=================================

Here are the goals of our bespoke rocking curve plan:

#. Make a plan that has a name indicative of the chore being
   accomplished.  This will be called ``rocking_curve()``.  At the
   bsui command line, it can be invoked simply as

   .. code-block:: python

      RE(rocking_curve())

#. Hide the motor and detector implementation details.

#. Make a live plot of the rocking curve as it is being measured.

#. Move to the correct DCM second crystal pitch position at the end of
   the scan.

#. Make changes to the data acquisition configuration to facilitate
   the measurement at the start of the scan and restore the
   configuration to a sensible resting state at the end.

#. Provide some on-screen feedback about the scan and the optimal
   pitch position.

Getting started
---------------

The rocking curve chore has fairly few variables.  It `always` moves
the DCM second crystal pitch and `always` measures the signal on the
:guilabel:`I0` detector.  Those simply are not parameters that the
user has to decide upon.  We can safely hide those details inside our
bespoke plan.

Here's our starting point:

.. code-block:: python
   :linenos:

   from bluesky.plans import rel_scan

   def rocking_curve(start=-0.10, stop=0.10, nsteps=101'):
       dets = [quadem1,]
       motor = dcm_pitch
       yield from rel_scan(dets, motor, start, stop, nsteps)
        
This already accomplishes the first and second objectives listed
above.  The motor and detector choices are made in the definition of
the plan and the plan is given an evocative name.

.. code-block:: python

   RE(rocking_curve())

Defaults are set for the scan range and the number of steps.  The
values of these defaults are chosen on the basis of experience at the
beamline and with the specific monochromator.  Of course if a staff
member or the user have a specific reason to scan over a longer range
or to take more or fewer steps, the defaults are easily overwritten,
like so:

.. code-block:: python

   RE(rocking_curve(start=-0.2, stop=0.3, nsteps=51))

There is one more detail about the detector that needs attention --
the integration time.  We want this scan to go fast.  It is not an
inherently interesting part of the user's experimental campaign.  Also
we know that the signal on :guilabel:`I0` when it is near the peak is
quite large.  So a very short integration time is quite adequate.

.. code-block:: python
   :linenos:

   from bluesky.plan_stubs import mv
   from bluesky.plans import rel_scan

   def rocking_curve(start=-0.10, stop=0.10, nsteps=101'):
       dets = [quadem1,]
       motor = dcm_pitch
       yield from mv(quadem.averaging_time, 0.1)
       yield from rel_scan(dets, motor, start, stop, nsteps)

The new line 7 sets the integration time to be $\frac{1}{10}$ second.


Adding a live plot
------------------

The implementation details of how live plots are handled at BMM is
beyond the scope of this example (and might merit it's own section
here in Bluesky by Example!), but I can show the overview of how it is
accomplished.

.. code-block:: python
   :linenos:

    from bluesky.plans import rel_scan

    def rocking_curve(start=-0.10, stop=0.10, nsteps=101'):
        dets = [quadem1,]
	motor = dcm_pitch
	yield from mv(quadem.averaging_time, 0.1)

        func = lambda doc: (doc['data'][motor.name], doc['data']['I0'])
        plot = DerivedPlot(func, xlabel=motor.name, ylabel='I0', 
	                   title='I0 signal vs. DCM 2nd crystal pitch')

        @subs_decorator(plot)	
	def scan_dcmpitch():
           yield from rel_scan(dets, motor, start, stop, nsteps)

        yield from scan_dcmpitch()

Lines 8 to 10 set up the live plot in the manner implemented at BMM.
`DerivedPlot
<https://github.com/NSLS-II-BMM/profile_collection/blob/master/startup/BMM/derivedplot.py>`__
is a fairly clunky tool used throughout BMM's profile.  It allows live
plots to show the ratios of signals, which is very commonly needed at
an XAFS beamline.  In this case, we are using it in a more trivial
way, just showing the signal on the :guilabel:`I0` detector.  This
could also be done with the `standard Bluesky LivePlot callback
<https://blueskyproject.io/bluesky/callbacks.html#liveplot-for-scalar-data>`__.

This plotting apparatus is then attached to a local function as a
function decorator at line 12.  The local function is called at
line 16.


Moving to the correct position
------------------------------

The plot is not actually the point of this plan.  The point is to find
the optimal pitch position and move the pitch to that position.

To do this, we look for the peak of the rocking curve.  That is, we
want to move to the ``dcm_pitch`` position for which the 
:guilabel:`I0` signal was maximized.  At that position, the lattice
planes of the two crystals are closest to parallel.

We add 

.. code-block:: python

   import pandas

to the top of the file defining this plan.  This imports the `pandas
<https://pandas.pydata.org/>`__ data analysis library, which is super
handy.  In particular we want the ``pandas.Series.idxmax`` function,
which will give us the index of the point in the :guilabel:`I0` signal
array containing the maximum value. [1]_  We then select the value of motor
position array at that index.  That is the peak position.

.. code-block:: python
   :linenos:

           uid = yield from rel_scan(dets, motor, start, stop, nsteps)	
	   t  = db[-1].table()
           signal = t['I0']
           position = pandas.Series.idxmax(signal)
           top = t[motor.name][position]
           yield from mv(motor, top)

In the last line, we move to the peak position, thus accomplishing the
goal of the plan.

Pre- and post-scan configuration changes
----------------------------------------

As the second crystal pitch is scanned over its rocking curve, the
height of the beam at the sample position changes with the pitch
position. Even though the angular change during the scan is tiny, the
sample is about 20 meters away from monochromator.

As a result, it is helpful to open the vertical hutch slits to give
the beam room to move around.  Once the optical pitch position is
found, we can close the slits back down to the operating size.

The other thing we want to consider is the integration time of the
detector.  This scan uses a quite short integration time, so it is
good practice to restore a resting-state value for that paraeter when
the scan is done.

These are a common enough sorts of chores that Bluesky provides a tool
exactly for this purpose.  It is called `finalize_wrapper
<https://nsls-ii.github.io/bluesky/generated/bluesky.preprocessors.finalize_wrapper.html>`__.
It works by defining two local functions within the
``rocking_curve()`` plan, like so:

.. code-block:: python
   :linenos:

    from bluesky.preprocessors import finalize_wrapper

    def rocking_curve(start=-0.10, stop=0.10, nsteps=101'):
        def main_plan(start, stop, nsteps):
	   ## (text of main plan)

        def cleanup_plan():
	   ## (text of cleanup plan)

        yield from finalize_wrapper(main_plan(start, stop, nsteps), cleanup_plan())

The two local plans are called in sequence by ``finalize_wrapper``.

Most of what we've discussed above will go into the text of the
``main_plan()``.  The ``cleanup_plan()`` will reclose the slits and
reset the integration time to its default value.

To flesh this out a bit:

.. code-block:: python
   :linenos:

    from bluesky.plan_stubs import mv
    from bluesky.preprocessors import finalize_wrapper

    def rocking_curve(start=-0.10, stop=0.10, nsteps=101'):
        def main_plan(start, stop, nsteps):
	    ## (text of main plan)

        def cleanup_plan():
            yield from mv(motor, slit_height)
	    yield from mv(quadem.averaging_time, 0.5)

	motor = dcm_pitch
	slit_height = slits3.vsize.readback.get()
	yield from mv(motor, 3)
        yield from mv(quadem.averaging_time, 0.1)
        yield from finalize_wrapper(main_plan(start, stop, nsteps), cleanup_plan())

With this plan definition, the preparatory tasks -- adjusting slit
height, adjusting integration time -- are done right before calling
``finalize_wrapper()``.  ``finalize_wrapper()`` is then called.
``main_plan`` does the bulk of the work and ``cleanup_plan`` restores
the beamline to its resting state.

There are two benefits to using ``finalize_wrapper`` to perform the
cleanup chores, rather than simply putting those chores at the end of
the main plan.  First, this pattern allows cleanup to be arbitrarily
complicated.  Second, and most importantly, the cleanup plan will get
run even if the main plan fails in some way.  This is a better (though
still not perfect) guarantee that the beamline will be left in a
sensible resting state.


Choosing the peak position
--------------------------

The version of this plan shown in the next section takes one more
argument beyond the start and stop positions and the number of steps.
The ``choice`` parameter is a string which tells the plan `how` to
find the peak position.  The default option is ``peak``, as discussed
above.  The other two options are to compute the center of mass of the
peak using a function imported from `SciPy <https://scipy.org/>`__ or
to perform a fit using `lmfit <https://lmfit.github.io/lmfit-py/>`__.
[2]_ The ``choice`` parameter would then be set to ``com`` or ``fit``,
respectively. 

Passing the ``choice`` parameter all the way into the ``main_plan()``
allows us to do this:

.. code-block:: python
   :linenos:

           uid = yield from rel_scan(dets, motor, start, stop, nsteps)
	   t  = db[-1].table()
           signal = t[sgnl]
           if choice.lower() == 'com':
               position = int(center_of_mass(signal)[0])
               top      = t[motor.name][position]
           elif choice.lower() == 'fit':
               pitch    = t['dcm_pitch']
               mod      = SkewedGaussianModel()
               pars     = mod.guess(signal, x=pitch)
               out      = mod.fit(signal, pars, x=pitch)
               print(out.fit_report(min_correl=0))
               out.plot()
               top      = out.params['center'].value
           else:
               position = pandas.Series.idxmax(signal)
               top      = t[motor.name][position]

           yield from mv(motor, top)

The peak position is computed by the selected algorithm, then the
motor is moved to that position.


On-screen feedback
------------------

Finally, it is helpful to provide some feedback to the bsui user as
this plan runs.  This is shown in lines 34, 61, and 63 in the full
code listing in the next section.  They are shown as simple
``print()`` statements, however those informative text strings can be
used in more interesting ways.  For example, that text could be
:ref:`posted to Slack <slack-text>` or used in some other interesting
way.


Plan composition
----------------

There is one final point to be made.  One of the great strengths of
Bluesky is its concept of plan composition.  In the example shown
here, everything is done using more primitive Bluesky plans,
specifically ``rel_scan()`` and ``mv()``, as well as
``finalize_wrpper()``.  

This concept of creating plans out of plans is very powerful because
it can be arbitrarily deep.  `Other` plans can be composed of the
``rocking_curve()`` plan.  Plans composed of plans composed of plans!

In fact, this is done at BMM.  The ``rocking_curve()`` plan is rarely
run on it's own.  It is occasionally useful to remeasure the rocking
curve by itself, but it is much more commonly done as part of a plan
which changes the configuration of the entire beamline according to
the element to be measured.

At BMM, this looks like this when run in bsui:

.. code-block:: python

   RE(change_edge('Fe'))

or, if the focusing mirror is to be used,

.. code-block:: python

   RE(change_edge('Zr', focus=True))

The algorithm for changing edges is a multi-step process which
includes the rocking curve measurement, but also movement of up to 14
motors, configuration of the fluorescence detector, and an
optimization scan of the position of the hutch slit assembly.

Plans composed of plans composed of plans.


Footnotes
---------

.. [1] This could also be done with `numpy.argmax
       <https://numpy.org/doc/stable/reference/generated/numpy.argmax.html>`__
       That said, if you are reading this document and do not know
       about `pandas <https://pandas.pydata.org/>`__, you should.
.. [2] Fitting could also be done using `SciPy's optimizer
       <https://docs.scipy.org/doc/scipy/reference/generated/scipy.optimize.curve_fit.html>`__. 
       That said, if you are reading this document and do not know
       about `lmfit <https://lmfit.github.io/lmfit-py/>`__, you should.
       
