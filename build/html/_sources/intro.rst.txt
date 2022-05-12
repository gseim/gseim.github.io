
============
Introduction
============

GSEIM (General-purpose Simulator with Explicit and Implicit
Methods) is meant for simulation of electrical circuits,
especially power electronic circuits, and also for numerical
solution of ordinary differential equations (ODEs). In terms of
basic functionality, GSEIM is similar to commercial packages such as
`Simulink <https://in.mathworks.com/products/simulink.html>`_,
`Simscape <https://in.mathworks.com/products/simscape.html>`_,
`PSIM <https://powersimtech.com/products/psim/capabilities-applications/>`_,
and
`PLECS <https://www.plexim.com/>`_.
However, at this stage, it does not handle real-time simulation.

The salient features of the GSEIM package can be described as follows.

- The solver, which handles the most intensive computation, viz.,
  numerical solution of circuit equations and ODEs, is written in
  C++ because of its high performance.
- For all other purposes, viz., schematic capture, parsing, parameter
  computation, and plotting, python is used because of the flexibility
  and ease of programming it offers.
- Output parameters, which determine what data gets stored in the output
  files during simulation, are specified without having to add extra
  elements, such as the *scope* in Simulink and
  PLECS, to the schematic diagram. This helps in avoiding clutter.
- Subcircuits (hierarchical blocks) can be used for simplifying the
  schematic.
- Explicit as well as implicit numerical methods are incorporated in the
  solver. Currently, the following methods are made available:

  - explicit (fixed time step): improved Euler, Heun, Runge-Kutta (4th order)
  - explicit (auto time step):
    RKF45, Bogacki and Shampine (2,3)
  - implicit (fixed time step):
    backward Euler, trapezoidal
  - implicit (auto time step):
    backward Euler-auto, trapezoidal-auto, TR-BDF2

  The backward Euler-auto and trapezoidal-auto methods are used
  only for nonlinear problems. In these methods, the time step is
  adjusted depending on the number of Newton-Raphson iterations
  required at a given time point.

- Steady-State Waveform (SSW) analysis option is made available,
  which is particularly useful for converter circuits where
  the steady-state solution is of interest.
- GSEIM provides a plotting GUI with the following features:

  - The user can control plot attributes such as line colour,
    line width, and symbol type.
  - In addition to basic plotting, some post-processing capabilties,
    viz., computation of average/rms values, Fourier coefficients,
    THD, are included.
  - The plotting GUI produces python code associated with the plot.
    If required, the user can edit this code in order to make the plot
    more suitable for a report or a presentation.

