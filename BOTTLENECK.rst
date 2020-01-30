Performance & Bottleneck Analysis
================================

The bottleneck analysis is a crucial step required for the performance optimization. Since it is stated that deterministic profiling can be highly precise, however, its overhead may affect its accuracy while statistical profiling has less overhead but lower precision in comparison, both profiling methods are applied.  The following tools and methods are applied to achieve the bottleneck analysis milestone.

Pyflame Statistical Profiler
--------
Pyflame is a statistical profiling tool capable of visualizing profiler output through interactive flame graph. Pyflame is implemented in C++, and uses the Linux ptrace system call to collect profiling information, it can profile program without modifying source code and it supports multi-threaded python programs. Pyflame has two distinct modes: by attaching to a running process, or tracing a command from start to finish.

CPU Profile for Local Trajectory Planner Module
--------
The conventional CPU profiling is done through the sampling of function calls stack traces. Below is the profile output of the running process 
``main_mod_local_traj.py``


.. raw:: html
    :file: mod_local_traj_interactive_profile2.svg 
    

From `Flame Graphs <http://www.brendangregg.com/flamegraphs.html>`_, the x-axis shows the stack profile population, sorted alphabetically (it is not the passage of time), and the y-axis shows stack depth, counting from zero at the bottom. Each rectangle represents a stack frame. The wider a frame is is, the more often it was present in the stacks. The top edge shows what is on-CPU, and beneath it is its ancestry. The colors are usually not significant, picked randomly to differentiate frames.

In other words, the stack traces are collected using sampling where each box represents the function. The boxes are stack from bottom (parent) to top (child) which corresponds to the calling ancestry. The horizontal ordering and colors has no indication of performance profile. The width of the stack boxes is proportional to the function time and frequency (for example, blocking time) according to the sample time.

In the graph, the module ``main_ltpl`` consumes most of the computation time, its main bottleneck children includes:

- ``OnlineTrajectoryHandler.py`` calling ``calc_splines.py`` from ``main_online_callback.py``
- ``OnlineTrajectoryHandler.py`` calling ``calc_vel_profile.py`` from ``trim_trajectory`` function.
- ``GraphBase.py`` called by ``gen_local_node_template.py`` (external library, could be optimized internally)
