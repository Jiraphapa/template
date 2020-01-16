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

With AOT there is no compilation overhead at runtime nor any overhead of importing Numba.
AOT compilation require to specify the function signatures explicitly (as shown in the hilighted line below), each exported function can have only one signature (can be multiple signature with different function names). On the other hand, the ``@jit`` decorators tells the compiler to compile the decorated function on-the-fly (at execution time) to produce efficient machine code, ``nopython=True`` is set to operate in ``nopython`` compilation mode that generates code that does not access the Python C API and the result of function compilation can be written into a file-based cache by passing ``cache=True`` to avoid compilation times each time you invoke a Python program (see `Numba's Glossary <https://numba.pydata.org/numba-doc/dev/glossary.html>`_).

.. code-block:: python
    :emphasize-lines: 1

    @cc.export('calc_vel_profile', 'float64[:](float64[:,:], float64[:], float64[:], boolean, float64, float64, optional(float64[:,:]), optional(float64[:,:]), optional(float64), optional(float64), optional(float64[:]), optional(float64), optional(float64), optional(int64))')
    @jit(nopython=True, cache=True)
    def calc_vel_profile(ax_max_machines: np.ndarray,
                     kappa: np.ndarray,
                     el_lengths: np.ndarray,
                     closed: bool,
                     drag_coeff: float,
                     m_veh: float,
                     ggv: np.ndarray = None,
                     loc_gg: np.ndarray = None,
                     v_max: float = None,
                     dyn_model_exp: float = 1.0,
                     mu: np.ndarray = None,
                     v_start: float = None,
                     v_end: float = None,
                     filt_window: int = None) -> np.ndarray:

The array types is declared by subscripting an elementary type according to the number of dimensions ex. ``float64[:]`` for 1-dimension double precision floating point (64 bit) array and ``float64[:,:,:]`` for 3-dimensions array, etc. (see `Numba's types and signatures <https://numba.pydata.org/numba-doc/dev/reference/types.html>`_). 
If you run this Python script, it will generate an extension module named ``calc_vel_profile``. Depending on the running platform, the actual filename may be ``calc_vel_profile.so``, ``calc_vel_profile.pyd``, ``calc_vel_profile.cpython-34m.so``, etc.

There are some limitations to the default parameters in currently used version of Numba (see this `thread <https://stackoverflow.com/questions/46123657/numba-calling-jit-with-explicit-signature-using-arguments-with-default-values>`_), in order to fulfill the functionality, one declares the ``optional(typ)`` decorator in the function signature indicating that optional type that allow any value of either of underlying typ or None.

Numba understands calls to NumPy ufuncs and is able to generate equivalent native code for many of them and NumPy arrays are supported as native types, however, not all Numpy implemenations are supported. The following code block of the function ``calc_vel_profile`` will thrown an error of `Use of unsupported NumPy function` in Numba compilation (see `Supported NumPy features <https://numba.pydata.org/numba-doc/dev/reference/numpysupported.html>`_).

.. code-block:: python

    # CASE 1: ggv supplied -> copy it for every waypoint
    if ggv is not None:
        p_ggv = np.repeat(np.expand_dims(ggv, axis=0), kappa.size, axis=0)

this can be solved by replacing with alternative implementation with supported Numpy features or writing code imposing Numpy implemenation:

.. code-block:: python

    # CASE 1: ggv supplied -> copy it for every waypoint
    if ggv is not None:
        p_ggv = np.empty((0, 0, 3))
        if kappa.size >= 0:
            p_ggv = np.expand_dims(ggv, axis=0)      # Notes: Numba 0.46.0 currently not support numpy.repeat with axis argument
            for i in range(kappa.size-1):            # same functionality with: p_ggv = np.repeat(np.expand_dims(ggv, axis=0), kappa.size, axis=0)
                p_ggv = np.concatenate((p_ggv, np.expand_dims(ggv, axis=0)), axis=0)    

Some parts are tricky, in the follwing hilighted lines would throw an error as ``loc_gg`` is an `optional` type which could be ``None``, an invalid parameter for ``np.column_stack`` function

.. code-block:: python
    :emphasize-lines: 3

    # CASE 2: local gg diagram supplied -> add velocity dimension (artificial velocity of 10.0 m/s)
    else:
         p_ggv = np.expand_dims(np.column_stack((np.ones(loc_gg.shape[0]) * 10.0, loc_gg)), axis=1)

The following version of ``calc_vel_profile`` implementation can be converted into the valid Numba version by adding ``np.copy`` to ensure the value of type ``ndarray`` as follows:

.. code-block:: python
    :emphasize-lines: 3

    # CASE 2: local gg diagram supplied -> add velocity dimension (artificial velocity of 10.0 m/s)
    else:
        p_ggv = np.expand_dims(np.column_stack((np.ones(loc_gg.shape[0]) * 10.0, np.copy(loc_gg))), axis=1)

todo: write about type unifying

.. code-block:: python
    :emphasize-lines: 11,13

    # ------------------------------------------------------------------------------------------------------------------
    # SEARCH START POINTS FOR ACCELERATION PHASES ----------------------------------------------------------------------
    # ------------------------------------------------------------------------------------------------------------------

    vx_diffs = np.diff(np.copy(vx_profile))
    acc_inds = np.where(vx_diffs > 0.0)[0]                  # indices of points with positive acceleration
    if acc_inds.size != 0:
        # check index diffs -> we only need the first point of every acceleration phase
        acc_inds_diffs = np.diff(acc_inds)
        acc_inds_diffs = insert(acc_inds_diffs, 0, 2)       # first point is always a starting point, Notes: Numba 0.46.0 currently not support numpy.insert 
        acc_inds_rel = acc_inds[acc_inds_diffs > 1]         # starting point indices for acceleration phases
    else:
        acc_inds_rel = [np.int64(x) for x in range(0)]      # if vmax is low and can be driven all the time

calc_splines_numba
------------



conv_filt_numba
----------




