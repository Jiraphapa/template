Guidelines for Further Development
================================

Potential Approach 
--------
As seen in the Performance Evaluation section, some modules can be optimized to achieve up to 50.9762% decrease in computation while another 
one may yield only 7.61145% improvement. With Numba alone the performance varied depend on the structure of the code and support for the Numpy function where 
Numba works best on code that uses NumPy arrays and functions, and loops. The efficiency improvement possibilities are


- Numpy support: Some Numpy implementations are not supported which leads to lower efficiency in optimization. Numba's community is maturing and in the future there's possibility for more Numpy support.


Frequent Issues and Troubleshooting
--------
Several issues and errors are encountered during the development and below are the approaches for commonly found errors: 
