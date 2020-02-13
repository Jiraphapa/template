Guidelines for Further Development
================================

Potential Approach 
--------
As seen in the Performance Evaluation section, some modules can be optimized to achieve up to 50.9762% decrease in computation while another 
one may yield only 7.61145% improvement. With Numba alone the performance varied depend on the structure of the code and support for the Numpy function where 
Numba works best on code that uses NumPy arrays and functions, and loops. The efficiency improvement possibilities may include


- Numpy support: some Numpy implementations are not supported which leads to lower efficiency in optimization, however, Numba's community is maturing and in the future there might be more support possibilities in Numpy implementation.
- SciPy support: apart from Numpy, SciPy is a free and open-source popular Python library also used for scientific computing and technical computing. SciPy usually contains more complex computation functionalities and modules and it is built on top of Numpy. This can 
be considered as an alternative in case of unsupported Numpy operation. Some operations in SciPy is claimed to be faster than Numpy in Numba (see `This thread <https://stackoverflow.com/questions/15670094/speed-up-solving-a-triangular-linear-system-with-numpy>`_), however, Numpy is generally faster than SciPy as it is written in C. Note 
that SciPy support in Numba is also limited.
- Cython:
- Parallel computing: 
- Compiler techniques:
- Usage of hardware accelerator:

tradeoffs

Frequent Issues and Troubleshooting
--------
Several issues and errors are encountered during the development and below are the approaches for commonly found errors: 
