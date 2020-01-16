Modules
================================

According to CPU profiling, the following Python mudules are the main bottleneck of the running process `main_mod_local_traj`:

1. **calc_vel_profile** 
  Calculates a velocity profile using the tire and motor limits as good as possible.
2. **calc_splines** 
  Solve for a curvature continuous cubic spline between given poses.
3. **conv_filt** 
  Filter a given temporal signal using a convolution (moving average) filter.

The above modules make a lot of use `NumPy` arrays and functions, and loops and in order to optimize these modules, the compiler techniques are considered for the optimal runtime speed. `Numba` is a just-in-time compiler for Python that works best on code that uses NumPy arrays and functions, and loops.
Each of the modules are converted into the optimized version with `Numba` in a separated modules named with the suffix `_numba` as follows:


calc_vel_profile_numba
--------
The ``calc_vel_profile_numba`` consists of the main functions ``calc_vel_profile`` (which usually called by external modules ex. ``OnlineTrajectoryHandler``) and ``calc_ax_poss``. The functions ``__solver_fb_unclosed``, ``__solver_fb_closed``, ``__solver_fb_acc_profile`` are used internally.
The first step is to import necessary Numba modules:

.. code-block:: python
   :emphasize-lines: 3,5

   from numba.pycc import CC
   from numba import jit

To produces a compiled extension module which does not depend on Numba, Numba's Ahead-of-Time compilation (AOT) allows to distribute the module on machines which do not have Numba installed (but Numpy is required). We declare the following module name with

.. code-block:: python
# Module name
cc = CC('calc_vel_profile_numba')

It is required to specify the function signatures explicitly, each exported function can have only one signature.

calc_splines_numba
------------



conv_filt_numba
----------




