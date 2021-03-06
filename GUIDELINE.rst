Guidelines for Further Development
================================

Potential Approach 
--------
As seen in the Performance Evaluation section, some modules can be optimized to achieve up to 50.9762% decrease in computation while another 
one may yield only 7.61145% improvement. With Numba alone the performance varied depend on the structure of the code and support for the Numpy function where 
Numba works best on code that uses NumPy arrays and functions, and loops. The efficiency improvement possibilities may include


- Numpy support
Some Numpy implementations are not supported which leads to lower efficiency in optimization, however, Numba's community is maturing and in the future there might be more support possibilities in Numpy implementation.

- SciPy support
Apart from Numpy, SciPy is a free and open-source popular Python library also used for scientific computing and technical computing. SciPy usually contains more complex computation functionalities and modules and it is built on top of Numpy. This can be considered as an alternative in case of unsupported Numpy operation. Some operations in SciPy is claimed to be faster than Numpy in Numba (see `this thread <https://stackoverflow.com/questions/15670094/speed-up-solving-a-triangular-linear-system-with-numpy>`_), however, Numpy is generally faster than SciPy as it is written in C. Note 
that SciPy support in Numba is also limited.

- Cython
Cython is a language with a mixture of C and Python syntax that aims to deliver the C performance on Python code. Unlike Numba which is a just-in-time (JIT) compiler that translates Python code into native machine instructions using the LLVM compiler library, Cython is translates Cython code to C code. Cython code is a separated file from Python file and compiled by C compiler. Cython is very matured and widely used in many scientific computing libraries. In comparison,
Numba is suitable for instantly integrate with existing project with minor code changes (if Numba code does not work, it can be revert back to Python easily). 

- Parallel computing
Numba supports parallelization both on CPU and GPU. The platform Nvidia Drive PX 2 using to run the algorithm and Numba support for both NVIDIA's CUDA provide a potential room for further development. Numba also supports SIMD vectorization which can automatically translate some loops into vector instructions for 2-4x speed improvements (see `Numba for CUDA GPUs <http://numba.pydata.org/numba-doc/latest/cuda/index.html>`_). 

It is inevitable to note the famous saying among software engineers ``“Premature optimization is the root of all evil”`` (Donald Ervin Knuth, The Art of Computer Programming, Volume 1: Fundamental Algorithms), too early optimization of the code is not a best practice and may lead to misfunctionality and spaghetti code. Optimization shall be 
done with code with possibly minimal further changes for effective delopment process.

Frequent Issues and Troubleshooting
--------
Several issues and errors are encountered during the development and below are the approaches for commonly found errors: 

- Cannot determine Numba type" when calling AOT-compiled function from AOT-compiled function
Problem when trying to call uncompiled function from a compiled function: ones shall compile the functions to be called from compiled function (see `this thread <https://github.com/numba/numba/issues/3823>`_).

- Cannot execute NumPy methods on non-contiguous arrays
Solution: replace numpy array with ``np.copy(numpy_array)`` (see `this thread <https://github.com/numba/numba/issues/1418>`_).

- Python(41093,0x7fff7623f300) malloc: *** error for object 0x7fea3b73c628: incorrect checksum for freed object - object was probably modified after being freed. #3
Caused by passing argument not the type declared (see `this thread <https://github.com/hhatto/otamapy/issues/3>`_).

- Untyped list problem
(see `untyped list troubleshooting <http://numba.pydata.org/numba-doc/latest/user/troubleshoot.html#my-code-has-an-untyped-list-problem>`_).

- Numba: calling jit with explicit signature using arguments with default values
How to deal with signatures of the function with default values 
(see `this thread <https://stackoverflow.com/questions/46123657/numba-calling-jit-with-explicit-signature-using-arguments-with-default-values>`_).

- Support for axis arguments on reduction functions
(see `this thread <https://github.com/numba/numba/issues/1269>`_).

- JIT results differs from AOT precompiled module
Caused by data type, precision (see `this thread <https://github.com/numba/numba/issues/2755>`_).

- None value error: None construct in nopython mode
(see `this thread <https://github.com/numba/numba/issues/3585>`_).

- Issue with np.concatenate
(see `this thread <https://github.com/numba/numba/issues/2787>`_).

- Tuple not supported
Tuple built-in is not supported in nopython mode (see `this thread <https://github.com/numba/numba/issues/2771>`_).

- numpy.hstack() not working in a jitted function
(see `this thread <https://stackoverflow.com/questions/54217007/numpy-hstack-not-working-in-a-jitted-function>`_).

- Issue with np.concatenate
(see `this thread <https://github.com/numba/numba/issues/2787>`_).

- Enable automatic parallel execution in pre-compiled code
(see `this thread <https://github.com/numba/numba/issues/3336>`_).

- Unicode string support
see the following solutions:

https://stackoverflow.com/questions/56463147/how-to-specify-the-string-data-type-when-using-numba

https://stackoverflow.com/questions/56463147/how-to-specify-the-string-data-type-when-using-numba

https://stackoverflow.com/questions/48987368/how-can-i-pass-string-type-in-class-in-numba-jitclass-python

https://stackoverflow.com/questions/32056337/python-can-numba-work-with-arrays-of-strings-in-nopython-mode

https://stackoverflow.com/questions/46708708/compare-strings-in-numba-compiled-function

https://github.com/numba/numba/issues/3323

https://github.com/numba/numba/issues/4018

https://github.com/numba/numba/pull/4425



