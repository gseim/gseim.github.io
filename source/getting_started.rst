
.. _getting_started:

===============
Getting Started
===============

It is assumed that the user has successfully installed GSEIM on a linux
computer. In this section, we will see how to run an existing GSEIM
project (which is already available in the GSEIM distribution), and
view the simulation results.

Free acceleration of induction motor
====================================

- File menu :math:`\rightarrow` Open
  :math:`\rightarrow` Select the **project file**
  ``~/gseim_grc/proj_grc/machines/indmc_free.grc``
  as shown below.

  .. image:: open_project.png
    :width: 600
    :alt: open project

  The project schematic diagram will appear as shown below.

  .. image:: indmc_schematic.png
    :width: 600
    :alt: demo project

- Click on ``Generate circuit file``. This would create the **circuit file**
  ``~/gseim_grc/gseim/output/indmc_free.in`` which is used by the GSEIM solver
  for solving the set of ODEs associated with the project schematic.
  Notice that a separate window opens to show the progress of this step.
  If the flow graph generation is successful, **Program Completed** will
  appear at the bottom of that window. Close the window.

- Click on ``Run simulation``. This invokes the solver which runs the
  simulation and creates the data files as specified for this project.
  Again, a separate window opens to show the progress made by the solver.
  The data files are created in the directory
  ``~/gseim_grc/gseim/output``.

- Click on ``View results``. The plotting GUI would show up as a separate
  application, as shown below.

  .. image:: plotting_gui.png
    :width: 600
    :alt: plotting GUI

  |
- Click on ``Load File``, and select the circuit file
  ``~/gseim_grc/gseim/output/indmc_free.in``. The data file created by GSEIM
  for this particular project will appear as shown below. Names of the
  variables stored in the data file are listed. Select ``time`` as
  the x-axis and ``wrm`` (the angular speed of the motor) as the y-axis to
  obtain the plot shown below.

  .. image:: indmc_results.png
    :width: 600
    :alt: indmc plot

Neutral point clamped inverter
==============================

- File menu :math:`\rightarrow` Open
  :math:`\rightarrow` Select the **project file**
  ``~/gseim_grc/proj_grc/dc_to_ac/npc_inverter.grc``.
  The project schematic diagram will appear as shown below.

  .. image:: npc_schematic.png
    :width: 600
    :alt: npc schematic

- Click on ``Generate circuit file``. After the **Program Completed** message
  appears, close the window, and click on ``Run simulation``.
  Close the window when the program is completed.

- Click on ``View results``. In the plotting GUI, click on ``Load File``,
  and select the circuit file ``~/gseim_grc/gseim/output/npc_inverter.in``.
  Select ``time`` as the x-axis and ``ia`` as the y-axis to
  obtain the plot shown below.

  .. image:: npc_results_1.png
    :width: 600
    :alt: npc results-1

  |
- We can view the Fourier components of the signal ``ia`` by clicking on
  ``Fourier`` in the plotting GUI and entering values for ``tStart`` and
  ``tEnd``, as shown below. Note that the difference between ``tStart``
  and ``tEnd`` is expected to be one period of the signal being analysed.

  .. image:: gui_fourier_1.png
    :width: 200
    :alt: fourier options

  |
- The Fourier components for ``ia`` will be displayed by the GUI.

  .. image:: npc_results_2.png
    :width: 600
    :alt: npc results-2


