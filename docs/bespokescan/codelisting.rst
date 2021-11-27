

.. _bespokescan_codelisting:

Code Listing
============

Here is the full listing of the rocking curve scan code.

An up-to-date version `can be found here
<https://github.com/NSLS-II-BMM/profile_collection/blob/master/startup/BMM/linescan.py>`__
at BMM's GitHub site.  This code listing is not identical to what is
found in the BMM repository.  The version in use at the beamline has a
few options which fall outside the scope of this tutorial.

The ``whisper()`` function at line 55 is one of the convenience
functions discussed in :ref:`colored_text`.


.. code-block:: python
   :linenos:

    from bluesky.plan_stubs import mv
    from bluesky.plans import rel_scan
    from bluesky.preprocessors import finalize_wrapper

    import numpy, pandas
    from lmfit.models import SkewedGaussianModel
    from scipy.ndimage import center_of_mass

    def rocking_curve(start=-0.10, stop=0.10, nsteps=101, choice='peak'):
        '''Perform a relative scan of the DCM 2nd crystal pitch around
        the current position to find the peak of the crystal rocking
        curve.  Begin by opening the hutch slits to 3 mm. At the end,
        move to the position of maximum intensity on I0, then return
        to the hutch slits to their original height.

        Parameters
        ----------
        start : (float)
            starting position relative to current [-0.1]
        end : (float)
            ending position relative to current [0.1]
        nsteps : (int)
            number of steps [101]
        choice : (string)
            'peak', fit' or 'com' (center of mass) ['peak']

        If choice is fit, the fit is performed using the
        SkewedGaussianModel from lmfit, which works pretty well for
        this measurement at BMM.  The line shape is a bit skewed due
        to the convolution with the slightly misaligned entrance
        slits.
        '''
        def main_plan(start, stop, nsteps, choice):
            func = lambda doc: (doc['data'][motor.name], doc['data']['I0'])
            dets = [quadem1,]
            sgnl = 'I0'
            titl = 'I0 signal vs. DCM 2nd crystal pitch'
            plot = DerivedPlot(func, xlabel=motor.name, ylabel=sgnl, title=titl)
    
            @subs_decorator(plot)
            def scan_dcmpitch(sgnl):
                line1 = f'{motor.name}, {sgnl}, {start:.3f}, {stop:.3f}, {nsteps} -- starting at {motor.user_readback.get():.3f}\n'
                        
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
                    print(whisper(out.fit_report(min_correl=0)))
                    out.plot()
                    top      = out.params['center'].value
                else:
                    position = pandas.Series.idxmax(signal)
                    top      = t[motor.name][position]
    
                print(f'rocking curve scan: {line1}\tuid = {uid}, scan_id = {user_ns['db'][-1].start['scan_id']}')
                yield from mv(motor, top)
		print(f'found optimal picth position at {motor.user_readback.get():.3f}')
            yield from scan_dcmpitch(sgnl)
    
        def cleanup_plan():
            yield from mv(slits3.vsize, slit_height)
            yield from mv(_locked_dwell_time, 0.5)
        
        motor = dcm_pitch
        yield from mv(_locked_dwell_time, 0.1)
        yield from mv(slits3.vsize, 3)
        slit_height = slits3.vsize.readback.get()
        yield from finalize_wrapper(main_plan(start, stop, nsteps, choice), cleanup_plan())
