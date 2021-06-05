
============
Introduction
============

GSEIM (General-purpose Simulator with Explicit and Implicit
Methods) is meant for solving a set of ordinary differential
equations (ODEs) arising from a system (circuit) schematic
diagram provided by the user. Its basic functionality may
be compared with the
`Matlab-Simulink <https://in.mathworks.com/products/simulink.html>`_
and
`Scicos <http://www.scicos.org/>`_
packages;
however, at this stage, GSEIM needs substantial enhancement
particularly in terms of library elements made available to
the user.

The salient features of the GSEIM package can be described as follows.

- The solver, which handles the most intensive computation, viz.,
  numerical solution of the ODEs, is written in C++ because of
  its high performance.
- For all other purposes, viz., schematic capture, parsing, parameter
  computation, and plotting, python is used because of the flexibility
  and ease of programming it offers.
- Output parameters, which determine what data gets stored in the output
  files during simulation, are specified without having to add extra
  elements, such as the *scope* in Simulink and
  Scicos, to the schematic diagram. This helps in avoiding clutter.
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

- GSEIM provides a GUI for plotting the variables specified by the
  user. The plotting GUI allows the user
  to select the output file of interest, the x-axis variable
  (typically time), and y-axis variable(s) to be included
  in the plot. It also gives the user control over plot attributes
  such as line colour, line width, and symbol type, with appropriate
  values specified by default. Using this information, the plotting
  GUI displays the plot requested by the user.
  It also produces python code associated
  with the plot. If required, the user can edit this code in order to make the plot
  more suitable for a report or a presentation. This way,
  the user can benefit from a wide range of plotting capabilities
  offered by python.

