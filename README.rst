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

and tell the interpreter to compile the module

.. code-block:: python

    if __name__ == "__main__":
        cc.compile()

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

There are some limitations to the default parameters in currently used version (0.46.0) of Numba (see this `thread <https://stackoverflow.com/questions/46123657/numba-calling-jit-with-explicit-signature-using-arguments-with-default-values>`_), in order to fulfill the functionality, one declares the ``optional(typ)`` decorator in the function signature indicating that optional type that allow any value of either of underlying typ or None.

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

One of the common reasons that of compile failure in Numba code is that it cannot statically determine the return type of a function. The above lines
will cause a `type unification` error as it cannot unify the `list` and `array` type (see `Numba's type unification problem <https://numba.pydata.org/numba-doc/latest/user/troubleshoot.html#my-code-has-a-type-unification-problem>`_)

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

This can be solved by explicitly casting the ``acc_inds_rel`` into `list` type, e.g. 

.. code-block:: python

    acc_inds_rel = list(acc_inds[acc_inds_diffs > 1])

calc_splines_numba
------------
The ``calc_splines_numba`` consists of the main functions ``calc_splines`` (which usually called by external modules ex. ``OnlineTrajectoryHandler``) and additional functions implementing Numpy features e.g. ``isclose``.
The first step is to import necessary Numba modules and declare module name, the additional `timeit` module can also be used for runtime measurement:

.. code-block:: python
   
   from timeit import Timer
   from numba.pycc import CC
   from numba import jit

   # Module name
   cc = CC('calc_vel_profile_numba')

AOT compilation require to specify the function signatures explicitly (as discussed in the `calc_vel_profile_numba` section).
The tuple return type is declared by the prefix ``UniTuple`` with the content type as the first parameter and number of elements as the second paramer. The function
``calc_splines`` returns a tuple of four double precision floating point (64 bit) arrays which are ``coeffs_x``, ``coeffs_y``, ``M`` and ``normvec_normalized``:

.. code-block:: python
    :emphasize-lines: 1

    @cc.export('calc_splines', 'UniTuple(float64[:,:],4)(float64[:,:], optional(float64[:]), optional(float64), optional(float64), optional(boolean))')
    @jit(nopython=True, cache=True)
    def calc_splines(path: np.ndarray,
                 el_lengths: np.ndarray = None,
                 psi_s: float = None,
                 psi_e: float = None,
                 use_dist_scaling: bool = True) -> tuple:

the function first check whether the path is close by calling ``np.isclose`` function, however, current version of Numba used (0.46.0) does not support the Numpy `isclose`, therefore,
the call is made to internal ``isclose`` function implementation.

.. code-block:: python
    :emphasize-lines: 19

    @cc.export('isclose', 'boolean[:](float64[:], float64[:])')
    @jit(nopython=True, cache=True)
    def isclose(a, b):  # implementation for np.isclose function
        assert np.all(np.isfinite(a)) and np.all(np.isfinite(b))
        rtol, atol = 1.e-5, 1.e-8
        x, y = np.asarray(a), np.asarray(b)
        # check if arrays are element-wise equal within a tolerance (assume that both arrays are of valid format)
        result = np.less_equal(np.abs(x-y), atol + rtol * np.abs(y))   
        return result 

    ..
    ..

    def calc_splines(...):

        ..
        ..

        if np.all(isclose(path[0], path[-1])):      # Numba 0.46.0 does not support NumPy function 'numpy.isclose'
            closed = True

The compact implemention of Numpy's is according to the original Numpy's implemention, with a restriction on finite array.

Another limitation of Numba is on the support of Numpy's ``diff`` function with ``axis`` argument, the default argument for the ``axis``
parameter the last axis, however, this can be fixed by array transposing technique.

.. code-block:: python
    :emphasize-lines: 8

    def calc_splines(...):

        ..
        ..

        # if distances between path coordinates are not provided but required, calculate euclidean distances as el_lengths
        if use_dist_scaling and el_lengths is None:
            #el_lengths = np.sqrt(np.sum(np.power(diff(path, axis=0), 2), axis=1))
            path_transpose = np.transpose(path)
            diff_path_transpose = np.diff(np.copy(path_transpose))
            diff_path_axis_0 = np.transpose(diff_path_transpose)
            el_lengths = np.sqrt(np.sum(np.power(diff_path_axis_0, 2), axis=1))

The unsupported Numpy ``diff`` operation in the row axis of ``path`` can be resolved by transposing the ``path`` array and calculate 
the discrete difference along the axis and then transpose back. Notes that there are lots of variables declaration due to 
requirement of static typing of AOT compilation mode (otherwise it may cause errors like in `BoundFunction` due to array reshape, incompatible size, etc.).

It is also important to note that with AOT compilation, Numba cannot statically determine the type of the variable, sometimes we need to 
explicitly cast the type, for example, in the step `create template for M array entries` of ``calc_splines`` function:

.. code-block:: python
    :emphasize-lines: 7,9

    b_x = np.zeros((no_splines * 4, 1))
    b_y = np.zeros((no_splines * 4, 1))

    ..
    ..

    b_x[j: j + 2] = np.array([[path[i,     0]],      # NOTE: the bounds of the two last equations remain zero
                         [path[i + 1, 0]]])
    b_y[j: j + 2] = np.array([[path[i,     1]],
                         [path[i + 1, 1]]])



It is explicitly specified with ``np.array(...)`` that the element declared inside is of type Numpy array, not a list of list. It is needed to be explicitly specified because the signature of ``b_x`` and ``b_y`` are Numpy array. This is also related to the `type unification` problem mentioned in ``calc_vel_profile_numba`` section.


conv_filt_numba
----------
The ``calc_splines_numba`` consists of the main functions ``conv_filt`` (which usually called within internal `trajectory planning helper` modules ex. ``calc_vel_profile``) and additional functions implementing Numpy helper features e.g. ``__get_middle_values``.
The first step is to import necessary Numba modules and declare module name just as above steps.

.. code-block:: python
   
   from timeit import Timer
   from numba.pycc import CC
   from numba import jit

   # Module name
   cc = CC('conv_filt_numba')

The main change from the original implemenation of ``conv_filt`` in ``conv_filt_numba`` is the apply convolution filter step:

.. code-block:: python
    :emphasize-lines: 4

    # apply convolution filter used as a moving average filter and remove temporary points
            signal_filt = np.convolve(signal_tmp,
                                    np.ones(filt_window) / float(filt_window),
                                    mode="same")[w_window_half:-w_window_half]

Another limitation of Numba is on the support of Numpy’s ``convolve`` function with ``mode`` argument, the default argument for the `mode` parameter is ‘full’, this returns the convolution at each point of overlap, with an output shape of (N+M-1,)
 where N is the first one-dimensional input array and M is the second one-dimensional input array, however, one shall implement the function for another `mode` support.

.. code-block:: python

    # returns output of length max(n1, n2).
    @cc.export('__get_middle_values', 'float64[:](float64[:], int64, int64)')
    @jit(nopython=True)
    def __get_middle_values(array, n1, n2):
        if n1 < n2:
            n1, n2 = n2, n1
        n = n2
        n_left = int(n/2)
        n_right = n - n_left - 1;
        return array[n_left:-n_right]

    ..
    ..

    def conv_filt(...):

        ..

        signal_filt = np.convolve(signal_tmp,
                                  np.ones(filt_window) / float(filt_window))

        # get_middle_values function works equivalent to adding 'mode="same"' argument in numpy.convolve
        signal_filt = __get_middle_values(signal_filt, signal_tmp.shape[0], filt_window)[w_window_half:-w_window_half]


The `mode = same` returns output of length max(M, N). The Numpy function ``convolve`` with mode ``same`` and ``full`` actually follows identical calculation step in implemenation, the 
only difference is the output length. The function ``__get_middle_values`` helps to trim the output from applying ``np.convolve`` into the output of length max(M, N).

