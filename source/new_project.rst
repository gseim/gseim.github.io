======================
Creating a new project
======================

It is assumed that you have successfully
installed GSEIM on a linux computer. In this section,
you will see how to make up a new GSEIM project,
simulate it, and view the simulation results.

Consider the ODE given by

.. math::

   \frac{dy}{dt} = 4 \cos \omega t + 3 \sin \omega t

This ODE can be implemented with a GSEIM schematic
by rewriting it as

.. math::

   x_1 = 4 \cos \omega t

   x_2 = 3 \sin \omega t

   y = \int (x_1 + x_2)\,dt

In the following, a step-by-step procedure is given to make
up the schematic diagram and obtain the simulation results
for the above equations. Note that this is a rather simple
ODE for which the analytic solution can be easily found and
compared with the numerical solution given by GSEIM.

- Start a new project. A canvas with an ``Options`` block will
  appear. Double-click on the options block, and set ``Id``
  and ``Title``. Note that Id should only contain letters, digits,
  and the underscore character. In the ``Generate option`` field,
  choose ``Circuit`` to indicate that this schematic diagram is
  for a complete stand-alone circuit (system) and not a hierarchical block.

  .. image:: new_project_1.png
    :width: 600
    :alt: Alternative text

  |
- The next step is to bring in the required elements (blocks)
  from the library. We need two ``src_ac`` (sinusoidal source)
  elements for
  :math:`x_1` and
  :math:`x_2`,
  a ``sum_2`` element and an ``integrator``. To bring ``src_ac``
  into the canvas, select it from the library, and drag it into
  the canvas.

  .. image:: new_project_2.png
    :width: 250
    :alt: Alternative text

  |
- Place the components, keeping in mind the connections you will
  need to make.

  .. image:: new_project_3.png
    :width: 350
    :alt: Alternative text

- Next, we need to set the properties (parameters) for each of the
  elements. In this example, the parameters for ``sum_2`` and
  ``integrator`` are not required to be changed (from their
  default values). For the ``src_ac`` elements, we need to set
  the amplitude, frequency, and phase. Double-click on the element
  of interest, and edit its parameter values. 
  For :math:`x_1 = 4 \cos \omega t`, the parameters
  ``a``, ``f_hz``, ``phi_deg``
  should be set to 4, 10, and 90, respectively.
  For :math:`x_2 = 3 \sin \omega t`, they
  should be set to 3, 10, and 0, respectively, as shown below.

  .. image:: new_project_4.png
    :width: 600
    :alt: Alternative text

  (Note that it is also possible to add comments about a block
  by double-clicking on it and then selecting the ``Advacned`` tab.)

- Connect the elements as required. To make a connection between two
  ports, click on one of the ports and then the other.

  .. image:: new_project_5.png
    :width: 350
    :alt: Alternative text

- We now want to indicate to the simulator which variables it should
  save (and make available for plotting). These are of two types:

  - Node value: This is the value of the variable represented by a
    node (wire). To select a node value as an output variable,
    left-click on the node (wire), and it will appear in a different
    colour as shown below.

    .. image:: new_project_6.png
      :width: 350
      :alt: Alternative text

    Now, right click on that wire and select ``Add to outvars``. A box
    which describes the node (as a connection between node x of element xx
    and node y of element yy) appears. Give a suitable name to this output
    variable as shown below.

    .. image:: new_project_7.png
      :width: 500
      :alt: Alternative text

    |
  - Output parameter of an element: For each element, a few output
    parameters are declared in the library. To select an output parameter
    of a specific element, right-click on that element and select
    ``Add to outvars``. The output parameters available for that element
    will appear. For example, if you click on the ``integrator``, the
    following dialog box will be displayed.
    Select ``y`` and give a suitable name to the corresponding output
    variable.

    .. image:: new_project_8.png
      :width: 400
      :alt: Alternative text

  |
- The next step is to set simulation parameters such as time step,
  choice of numerical method, etc. This is done by preparing one or
  more ``solve blocks``. In most cases, including the present example,
  we only need one solve block.

  To begin with, we need to add a new solve block. Click on
  ``Edit``
  :math:`\rightarrow`
  ``SolveBlocks``
  :math:`\rightarrow`
  ``Add Solve Block``.
  A dialog box for a new solve block appears as shown below.
  Choose a suitable name for this solve block. The ``index`` of
  this block can keep its default value.

  .. image:: new_project_9.png
    :width: 250
    :alt: Alternative text

  We have now added a solve block called ``S1(0)``.

- Next, we edit the properties of the solve block ``S1(0)``. Click on
  ``Edit``
  :math:`\rightarrow`
  ``SolveBlocks``
  :math:`\rightarrow`
  ``Edit Solve Block``.
  A dialog box with several parameters appears. For the present example,
  we only need to set the following.

  - numerical method: We can choose Runge-Kutta order 4 (RK4), for example.
  - ``t_start`` and ``t_end``, the starting and ending times for the simulation
  - the time step ``tstep0_x``

  |

  .. image:: new_project_10.png
    :width: 400
    :alt: Alternative text

  |
- We now need to prepare an ``output block`` to inform the simulator
  about what data files should be created and which variables should
  be stored in each of them. For this purpose, we first need to add
  an output block.  Click on
  ``Edit``
  :math:`\rightarrow`
  ``OutputBlocks``
  :math:`\rightarrow`
  ``Add Output Block``.
  The following dialog box appears.

  .. image:: new_project_11.png
    :width: 200
    :alt: Alternative text

  Each output block is associated with a parent solve block.
  In this case, there is only one solve block, so there is no need to
  exercise any choice.

- Edit the output block added in the previous step by clicking on
  ``Edit``
  :math:`\rightarrow`
  ``OutputBlocks``
  :math:`\rightarrow`
  ``Edit Output Block``.
  The following dialog box appears.

  .. image:: new_project_12.png
    :width: 450
    :alt: Alternative text

  Note that the output variables listed in this box are those declared
  earlier. Select the output variables of interest, and assign a name to
  the output file (which will be created by this output block).

- You are now ready to simulate the system. Save the project
  file in the directory of your choice, naming it
  ``test_1.grc`` (where ``test_``` comes from the ``Id`` in the
  ``Options`` block). Click on
  ``Generate flow graph``, then
  ``Execute flow graph``.
  At this stage, the data file specified in your output block would
  be created in the directory
  ``~/gseim_gui/gseim/output/``. Click on ``View results``. The plotting
  GUI will appear in a separate window. Click on ``Browse files`` and select
  ``~/gseim_gui/gseim/output/test_1.in``. After that, select the x and y variables
  as shown in the following figure to view :math:`y(t)`.

  .. image:: new_project_13.png
    :width: 650
    :alt: Alternative text

