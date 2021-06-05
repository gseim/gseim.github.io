============
Solve Blocks
============

After a circuit schematic is prepared, we need to
convey to the simulator what kind of simulation needs
to be performed, which method to use, which output
parameters should be saved, etc. This task is performed
by a solve block. Solve blocks are of two types:

- ``startup`` solve block for :ref:`start-up simulation <startup>` 
- ``trns`` solve block for transient simulation

Multiple solve blocks per project are allowed. Here are
a few situations where multiple solve block would be useful:

- We want to perform a start-up simulation and use the
  solution as a starting point for transient simulation.

- We have performed transient simulation from
  :math:`t = 0` to
  :math:`t = t_1`.
  We want to continue from
  :math:`t_1`
  and perform transient simulation up to
  :math:`t_2`,
  without having to repeat the
  :math:`0 < t < t_1`
  part.

In most cases, we are interested in a single transient simulation,
and in that case, only one solve block is required.

A new solve block can be added by clicking on
``Edit``
:math:`\rightarrow`
``SolveBlocks``
:math:`\rightarrow`
``Add Solve Block``.

A dialog box for a new solve block appears as shown below.

.. image:: new_project_9.png
  :width: 250
  :alt: Alternative text

The solve block needs to be assigned a suitable name, e.g, ``S1``.
The ``index`` field determines the order in which solve blocks are
executed. For example, if there are two solve blocks with names
``S1``, ``S2`` and indices 0, 1, respectively, the solve block
``S1(0)`` will be executed first, followed by ``S2(1)``.

When a solve block is added, it gets a set of default parameter values
from the file ``~/gseim_gui/gseim/exec/gseim_slvparms.in``.
These values can be edited by clicking on
``Edit``
:math:`\rightarrow`
``SolveBlocks``
:math:`\rightarrow`
``Edit Solve Block``.

A dialog box with parameters appears as shown below.

.. image:: new_project_10.png
  :width: 400
  :alt: Alternative text

We will now describe the solve block parameters.
The following points may be noted.

- Keeping future developments in mind, some of the parameters
  have been given names starting with ``x_`` (or ending with ``_x``),
  which means that they are related to ``xbe``'s as opposed to ``ebe``'s
  (electrical basic elements) to be introduced in the near future.
- Parameters with ``_nr_`` in their names are related to the Newton-Raphson
  method.

Solve Block Statements
======================

- ``solve_type``: *trns* for transient simulation, *startup* for
  start-up simulation

- ``initial_sol`` is related to the initial solution, i.e., the starting
  point for transient simulation. There are three options.

  - *initialize* for initializing variables before transient simulation
  - *previous* for continuing with the solution obtained in the previous
    solve block as the initial solution
  - *read_from_file* for reading the initial solution from a file

- ``initial_sol_file``: name of the solution file (This field is relevant
  if ``initial_sol`` is *read_from_file* 

- ``output_solution_file``: name of the solution file (needs to be supplied
  only if the user wants to save the solution file)

- ``algorithm_x_trns``: transient simulation algorithm

- ``algorithm_x_startup``: start-up simulation algorithm

- ``itmax_trns``: maximum number of time points allowed in transient simulation
  This is a *safety feature*. If the user mistakenly creates conditions which
  lead to a huge number of time points, the output files could fill up the
  disk

- ``t_start``: starting time for transient simulation

- ``t_end``: ending time for transient simulation

- ``tstep0_x``: time step. For constant time-step methods, this parameter specifies
  the uniform time interval. For auto time-step methods, it specifies the initial
  time step
  (see
  `SEQUEL User's manual-Part 1 <https://www.ee.iitb.ac.in/~sequel/sequel_manual_1.pdf>`_)

- ``delt_min_x``: smallest time step allowed during transient simulation

- ``delt_max_x``: largest time step allowed during transient simulation

- ``t_startup``: time at which start-up simulation is to be performed (ignored
  for transient simulation)

- ``factor_step_increase``: factor by which time step should be increased
  in successive iterations in auto time-step methods.

- ``factor_step_decrease``: factor by which time step should be decreased
  in successive iterations in auto time-step methods.

- ``rkf45_tolr``: tolerance for the RKF45 method

- ``rkf45_fctr_min``: lower limit on multiplier :math:`k` of RKF45 algorithm
  for time step change, where
  :math:`\Delta t_{\mathrm{new}} = k \times \Delta t_{\mathrm{new}}`.

- ``rkf45_fctr_max``: upper limit on multiplier :math:`k` of RKF45 algorithm

- ``bs23_tolr``: tolerance for the BS23 method

- ``bs23_fctr_min``: lower limit on multiplier :math:`k` of BS23 algorithm
  for time step change

- ``bs23_fctr_max``: upper limit on multiplier :math:`k` of BS23 algorithm

- ``trbdf2_tolr``: tolerance for the TR-BDF2 method

- ``itmax_trbdf2``: maximum number of successive reductions in the time step at
  a given time point when TR-BDF2 algorithm is selected.

- ``itmax_stepred``: maximum number of successive reductions in the time step at
  a given time point when an auto time-step algorithm (other than TR-BDF2) is selected.

- ``write_time_x``: GSEIM writes the iteration number to the console during
  transient simulation to indicate the progress of the solver. If the time should
  also be written, this parameter should be set to *yes*.

- ``write_iter_n_x``: Interval (in terms of iterations, not time) after which
  GSEIM should write the iteration number to the console.

- ``write_matrix_x``: meant for debugging; ignore.

- ``x_nr_itermax``: maximum number of Newton-Raphson (NR) iterations

- ``x_nr_dmp``: *yes* if damping should be used in NR iterations
  (see
  `SEQUEL User's manual-Part 1 <https://www.ee.iitb.ac.in/~sequel/sequel_manual_1.pdf>`_)

- ``x_nr_dmp_itermax``: number of NR iterations for which damping should be applied

- ``x_nr_dmp_k``: damping factor for NR iterations

- ``x_nr_itermax0``, ``x_nr_dmp0``, ``x_nr_dmp_itermax0``, ``x_nr_dmp_k0``: like
  the above NR parameters except that these apply to the first time point.

- ``x_nr_check_rhs2``: *yes* if the 2-norm should be used to check for convergence
  (see
  `SEQUEL User's manual-Part 1 <https://www.ee.iitb.ac.in/~sequel/sequel_manual_1.pdf>`_)

- ``x_nr_write_rhs2``: *yes* for writing the 2-norm to the console (used for debugging)

- ``x_nr_eps_rhs``: tolerance value for the 2-norm

- ``delt_small``: parameter used in comparing time points. It can be
  set to two to three orders of magnitude smaller than ``delt_min``.

- ``x_fixed_tstep``: for future development; ignore.

