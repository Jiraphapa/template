========
Usage
========

To import the compiled module, make sure to add the module_name.so file to PYTHONPATH before import:

.. code-block:: python

	import module_name

To invoke the functions, simply call:

.. code-block:: python

	import module_name.function_name_as_declared_in_signature

For example, to call ``calc_splines`` function in the module ``calc_splines_numba``:

.. code-block:: python
	:emphasize-lines: 7

	import calc_splines_numba
	import numpy as np 

	path = np.ones((15,2))
	el_lengths = np.ones((14))

	calc_splines_numba.calc_splines(path, el_lengths, None, None, None)
