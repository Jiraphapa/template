Modules
================================

According to CPU profiling, the following mudules are the main bottleneck of the running process `main_mod_local_traj`:
1. **calc_vel_profile** <br />
  Calculates a velocity profile using the tire and motor limits as good as possible.
2. **calc_splines** <br />
  Solve for a curvature continuous cubic spline between given poses.
3. **conv_filt** <br />
  Filter a given temporal signal using a convolution (moving average) filter.



calc_vel_profile
--------

- Be awesome
- Make things faster

calc_splines
------------

Install $project by running:

    install project

conv_filt
----------

- Issue Tracker: github.com/$project/$project/issues
- Source Code: github.com/$project/$project

Support
-------

If you are having issues, please let us know.
We have a mailing list located at: project@google-groups.com

License
-------

The project is licensed under the BSD license.
