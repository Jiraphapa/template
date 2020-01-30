Performace Evaluation
================================

The measurement of the performance is done by using the ``timeit`` module functions. As Numba has to compile the function for the argument types given before it executes the machine code version of the function, there can be overhead in compilation time when measuring 
calling function for the first time but when it is called again the with same types, it can reuse the cached version instead of having to compile again (setting ``cache=True`` in ``@jit`` decorator), results 
in a faster code. For instance, the code taken partially from ``calc_splines_numba.py``:

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

    # testing -------------------------------
    if __name__ == "__main__":
        pass

    path = np.ones((15,2))
    el_lengths = np.ones((14))
    t = Timer(lambda: calc_splines(path,el_lengths))

    print("Execution time for calc_splines with numba (with compilation):",t.timeit(number=1))
    print("Execution time for calc_splines with numba (after compilation):",t.timeit(number=1))


The example code are executed on x86\_64 architecture and gave the following output:

.. code-block:: python

    Execution time for calc_splines with numba (with compilation): 3.6058600189999988
    Execution time for calc_splines with numba (after compilation): 0.0004452480000001202

Comparing with original ``calc_splines.py`` where the execution time was  0.0011502260000000042 seconds, Numba optimized code can boost the execution time to approximately 2.58 times faster on the given input. To follwing steps are applied to measure the execution time (The ``timeit`` tool can also be invoke in the command-line):

- importing ``from timeit import Timer``
- define a timer for function call like ``t = Timer(lambda: calc_splines(path))``
- item execute timer ``t.timeit(number=1)`` multiple times and compare the time of first call and last execution



According to CPU profiling, the following Python mudules are the main bottleneck of the running process `main_mod_local_traj`:

1. **calc_vel_profile** 
  Calculates a velocity profile using the tire and motor limits as good as possible.
2. **calc_splines** 
  Solve for a curvature continuous cubic spline between given poses.
3. **conv_filt** 
  Filter a given temporal signal using a convolution (moving average) filter.

Each of the modules are converted into the optimized version with Numba in a separated modules named with the suffix `_numba`.

Exucution Time Comparison
--------

The contained data used to test is always for an entire race track (Berlin, Monteblanco, Modena). Below are comparisons of execution time of the original and optimized modules. 

.. list-table:: Execution time comparison for `calc_vel_profile` and `calc_vel_profile_numba` module
   :widths: 25 25 50
   :header-rows: 1

   * - Module name
     - Average execution time (without compilation)
   * - calc_vel_profile
     - 55
   * - calc_vel_profile_numba
     - 555

