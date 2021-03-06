Performace Evaluation
================================

The timeit Module for Execution Time Measurement
--------
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

Testing With Race Track (Berlin, Monteblanco, Modena) Data
--------
To evaluate the functions given the `Pickle` file of race track data, ones shall specify the path and load files with ``.pkl`` extension. 
In the example below, the dictionary data of each race track is stored in the ``inputs`` list. The code below outputs the execution time before and after the compilation.

.. code-block:: python

  from timeit import Timer
  import pickle
  import os, glob

  path = 'unittest/calc_splines_inputs/'
  inputs = list()
  for filename in glob.glob(os.path.join(path, '*.pkl')):
      with open(filename, 'rb') as fh:
          data = pickle.load(fh)
          inputs.append(data)

  input = inputs[0]
  print("Execution time for calc_splines with numba (with compilation):",t.timeit(number=1))

  for input in self.inputs:
    path, el_lengths, psi_s, psi_e, use_dist_scaling = input['path'], input.get('el_lengths',None), input.get('psi_s',None), input.get('psi_e',None), input.get('use_dist_scaling',True)
    t = Timer(lambda: calc_splines(path, el_lengths, psi_s, psi_e, use_dist_scaling))
    print("Execution time for calc_splines with numba (after compilation):",t.timeit(number=1))

To get an average execution time, ones could use ``timeit.repeat`` to benchmark repeatedly. The default repetition is 3 and the parameter `n` is the number of executions.

.. code-block:: python

  reps = timeit.repeat(repeat=3, n=10000,
                      stmt="calc_splines(path, el_lengths, psi_s, psi_e, use_dist_scaling)",
                      setup="import numpy as np")

  average_time = np.mean(reps)

Further examples on unit testing can be found on the unittest modules in the repository.
  

Exucution Time Comparison for Numba-optimized modules
--------
The contained data used to test is always for an entire race track (Berlin, Monteblanco, Modena). Below are comparisons of execution time of the original and optimized modules. The system is tested on the x86_64 platform 2,4 GHz Intel Core i5 with 
memory 16 GB.

.. list-table:: Table 1: Execution time comparison
   :widths: 40 40
   :header-rows: 1

   * - Module name
     - Average execution time after compilation (seconds)
   * - calc_vel_profile.py
     - 0.0003124909999999981
   * - calc_vel_profile_numba.py
     - 0.00015319499999977282

The module ``calc_vel_profile_numba`` achieved an average of 50.9762% decrease in computation time.

.. list-table:: Table 2: Execution time comparison
   :widths: 40 40
   :header-rows: 1

   * - Module name
     - Average execution time after compilation (seconds)
   * - calc_splines.py
     - 0.4699571319999999
   * - calc_splines_numba.py
     - 0.43418659700000006

The module ``calc_splines_numba`` achieved an average of 7.61145% decrease in computation time.

    



