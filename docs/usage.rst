========
Usage
========

To import the compiled module, make sure to add the module_name.so file to PYTHONPATH and import::

	import module_name

To invoke the functions, simply call::

	import module_name.function_name_as_declared_in_signature

For example, to call ``calc_splines`` function in the module ``calc_splines_numba``::

	import calc_splines_numba
