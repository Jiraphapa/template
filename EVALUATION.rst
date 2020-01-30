Performace Evaluation
================================

The measurement of the performance is done by using the ``timeit`` module functions. As Numba has to compile the function for the argument types given before it executes the machine code version of the function, there can be overhead in compilation time when measuring calling function for the first time but when it is called again the with same types, it can reuse the cached version instead of having to compile again (setting ``cache=True`` in ``@jit`` decorator), results in a faster code. For instance, the code taken partially from ``calc_splines_numba.py``::

.. code-block:: python

    import numpy as np
    import math
    from timeit import Timer
    from numba import jit

    @jit(nopython=True, cache=True)
    def calc_splines(path: np.ndarray,
                    el_lengths: np.ndarray = None,
                    psi_s: float = None,
                    psi_e: float = None,
                    use_dist_scaling: bool = True) -> tuple:
                    
    ...
    ...
    ...

    # testing -------------------------------
    if __name__ == "__main__":
        pass

    path = np.ones((15,2))
    el_lengths = np.ones((14))
    t = Timer(lambda: calc_splines(path,el_lengths))
    print("Execution time for calc_splines with numba (with compilation):",t.timeit(number=1))

    print("Execution time for calc_splines with numba (after compilation):",t.timeit(number=1))


According to CPU profiling, the following Python mudules are the main bottleneck of the running process `main_mod_local_traj`:

1. **calc_vel_profile** 
  Calculates a velocity profile using the tire and motor limits as good as possible.
2. **calc_splines** 
  Solve for a curvature continuous cubic spline between given poses.
3. **conv_filt** 
  Filter a given temporal signal using a convolution (moving average) filter.

Each of the modules are converted into the optimized version with `Numba` in a separated modules named with the suffix `_numba` as follows:


calc_vel_profile_numba
--------
