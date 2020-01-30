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

.. image:: images/mod_local_traj_interactive_profile.svg.svg
  :width: 600
  :alt: Flame graph of the process main_mod_local_traj.py

.. raw:: html

   <img src="mod_local_traj_interactive_profile.svg" alt="Flame graph of the process main_mod_local_traj.py></a>


The stack traces are collected using sampling where each box represents the function. The boxes are stack from bottom (parent) to top (child) which corresponds to the calling ancestry. The horizontal ordering and colors has no indication of performance profile. The width of the stack boxes is proportional to the function time and frequency (for example, blocking time) according to the sample time.

In the graph, the module ``main_ltpl`` consumes most of the computation time, its main bottleneck children include ``OnlineTrajectoryHandler.py``, ``main_online_callback.py``, ``calc_splines.py``, ``gen_local_node_template.py``, ``calc_vel_profile.py``, ``GraphBase.py`` and so on (see detailed analysis by investigating interactive image above).