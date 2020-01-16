Modules
================================

According to CPU profiling, the following mudules are the main bottleneck of the running process `main_mod_local_traj`:

1. **calc_vel_profile** <br />
  Calculates a velocity profile using the tire and motor limits as good as possible.
2. **calc_splines** <br />
  Solve for a curvature continuous cubic spline between given poses.
3. **conv_filt** <br />
  Filter a given temporal signal using a convolution (moving average) filter.
Each of the following modules are converted into the optimized version with Numba in a separated modules named with the suffix `_numba` as follows:


calc_vel_profile_numba
--------
The `calc_vel_profile` is 


calc_splines_numba
------------



conv_filt_numba
----------




