
.. _xbe:

===========================
Element templates (``xbe``)
===========================

A very important feature of GSEIM is that it allows the user to
add new functionality in the form of library elements, by
writing suitable **templates**.
In this section, we look at the syntax of element templates
with the help of some examples. We start with a few remarks.

- Element templates are files with extension ``.xbe``
  (*explicit basic element*) in ``~/gseim_gui/gseim/xbe/``.

- An element template has three types of variables in general:
  input, output, and auxiliary. Only the input and output variables
  are made available in the schematic capture GUI for connection to
  other elements.

- Two types of elements are allowed:

  - ``evaluate`` type in which the element equations are of the form
    :math:`y = f(x_1,x_2,..)`
    where :math:`y` is an output and
    :math:`x_1`,
    :math:`x_2`, etc.
    are inputs. These elements do not involve time derivatives.

  - ``integrate`` type with equations of the form
    :math:`\displaystyle\frac{dy}{dt} = f(x_1,x_2,..)` where
    :math:`y` is an
    output or auxiliary variable, and
    :math:`x_1`,
    :math:`x_2`, etc.
    can be input, output, or auxiliary variables.

``xbe`` template examples
=========================

In the following, we look at a few representative examples
to explain the functioning of the ``xbe`` templates. The
templates described here can be found in
``~/gseim_gui/gseim/xbe``.

.. _sum_2:

``sum_2.xbe``
-------------

This element is used to obtain

.. math::

   y = k_1x_1 + k_2x_2,

where
:math:`x_1`,
:math:`x_2` are input variables,
:math:`y` is the output variable, and
:math:`k_1`,
:math:`k_2` are real parameters.
This is an ``evaluate`` type element, i.e., its output can be written
as a function of its inputs, and it does not involve time derivatives.
The overall structure of ``sum_2.xbe`` is given below.

.. code-block:: text
   :linenos:

   xbe name=sum_2 evaluate=yes
   # y = k1*x1 + k2*x2
   Jacobian: constant
   input_vars: x1 x2
   output_vars: y
   aux_vars:
   iparms:
   sparms:
   rparms: k1=1 k2=1
   stparms:
   igparms:
   outparms: x1 x2 y
   n_f= 0
   n_g= 1
   g_1: x1 x2 y
   C:
      k1 = X.rprm[nr_k1];
      k2 = X.rprm[nr_k2];
      if (G.flags[G.i_init_guess]) {
        X.val_vr[nvr_y] = k1*X.val_vr[nvr_x1] + k2*X.val_vr[nvr_x2];
        return;
      }
      if (G.flags[G.i_trns] || G.flags[G.i_startup]) {
        if (G.flags[G.i_explicit]) {
          X.val_vr[nvr_y] = k1*X.val_vr[nvr_x1] + k2*X.val_vr[nvr_x2];
        } else if (G.flags[G.i_implicit]) {
          if (G.flags[G.i_function]) {
            X.g[ng_1] = X.val_vr[nvr_y]
              - k1*X.val_vr[nvr_x1] - k2*X.val_vr[nvr_x2];
          }
          if (G.flags[G.i_jacobian]) {
            J.dgdvr[ng_1][nvr_y ] =  1.0;
            J.dgdvr[ng_1][nvr_x1] = -k1;
            J.dgdvr[ng_1][nvr_x2] = -k2;
          }
        }
        return;
      }
      if (G.flags[G.i_outvar]) {
        X.outprm[no_x1] = X.val_vr[nvr_x1];
        X.outprm[no_x2] = X.val_vr[nvr_x2];
        X.outprm[no_y ] = X.val_vr[nvr_y ];
        return;
      }
   endC
   endxbe

Note the following features in the template:

- The element name is specified by the keyword ``name``.
- ``evaluate=yes`` specifies the element type.
- A line starting with ``#`` is a comment.
- ``Jacobian: constant`` indicates that,
  when the element equation
  :math:`y - k_1x_1 - k_2x_2 = 0`
  is differentiated with respect to the variables involved
  in the equation, we get constants.
- The lines ``input_vars`` and ``output_vars`` specify
  the input and output variables of the element, respectively.
- The names and default values of the real parameters are given
  by the ``rparms`` statement. (GSEIM also allows integer and
  string parameters; they are not used in ``sum_2``).
- The ``outparms`` statement specifies the names of output
  parameters which will be made available by this template
  during simulation (if requested by the user).
- The ``n_f`` and ``n_g`` statements specify the number
  of ``f`` and ``g`` functions for this element. (This aspect will be
  described in the :ref:`Functions <functions>` section.)
- The ``g_1`` statement indicates the variables involved in
  the function
  :math:`g_1`.
- The C++ section of the template appears
  between the ``C`` and ``endC`` statements.

The behaviour of this element is coded in the C++
section of the template.  In order to understand this section, we need to
see where it fits in the overall scheme, as explained in the following.

There is a GSEIM library preprocessor which picks up the C++ section
of each ``xbe`` template (as also the other details such as input
and output variables, real parameters, etc.) and uses it to prepare
a C++ routine for that specific ``xbe``. All of these ``xbe``
routines are then compiled together with the solver code of GSEIM
to prepare the executable file for the solver.

The ``sum_2`` routine (function) receives objects ``G``
and ``X`` from the GSEIM main program and is expected to compute
various quantities such function values, output parameters, etc.
Objects ``G`` and ``X`` may be described as follows.

- ``G`` is a global object and is used to pass information
  about the current time point, type of method being used (implicit or
  explicit), etc. It also conveys to the element routine, through the
  ``flags`` array, what computation the main program is expecting
  from the element routine in the present call.
- ``X`` is specific to the element being treated, and it contains
  variables and parameter values related to that element.

With this background, let us now look at the C++ routine
prepared by the library preprocessor for ``sum_2.xbe``:

.. code-block:: text
   :linenos:

   void x_sum_2(Global &G,XbeUsr &X,XbeJac &J) {
      double x1,x2;
      double y;
      double k1,k2;
      const int nvr_x1 = 0;
      const int nvr_x2 = 1;
      const int nvr_y = 2;
      const int nr_k1 = 0;
      const int nr_k2 = 1;
      const int no_x1 = 0;
      const int no_x2 = 1;
      const int no_y = 2;
      const int ng_1 = 0;
      k1 = X.rprm[nr_k1];
      k2 = X.rprm[nr_k2];
      if (G.flags[G.i_init_guess]) {
        X.val_vr[nvr_y] = k1*X.val_vr[nvr_x1] + k2*X.val_vr[nvr_x2];
        return;
      }
      if (G.flags[G.i_trns] || G.flags[G.i_startup]) {
        if (G.flags[G.i_explicit]) {
          X.val_vr[nvr_y] = k1*X.val_vr[nvr_x1] + k2*X.val_vr[nvr_x2];
        } else if (G.flags[G.i_implicit]) {
          if (G.flags[G.i_function]) {
            X.g[ng_1] = X.val_vr[nvr_y]
              - k1*X.val_vr[nvr_x1] - k2*X.val_vr[nvr_x2];
          }
          if (G.flags[G.i_jacobian]) {
            J.dgdvr[ng_1][nvr_y ] =  1.0;
            J.dgdvr[ng_1][nvr_x1] = -k1;
            J.dgdvr[ng_1][nvr_x2] = -k2;
          }
        }
        return;
      }
      if (G.flags[G.i_outvar]) {
        X.outprm[no_x1] = X.val_vr[nvr_x1];
        X.outprm[no_x2] = X.val_vr[nvr_x2];
        X.outprm[no_y ] = X.val_vr[nvr_y ];
        return;
      }
      return;
   }

Note that the library preprocessor has simply inserted
the C++ section of ``sum_2.xbe`` into this routine
without any changes. In addition, it has added the following.

- declaration for ``x1``, ``x2``, ``y``, ``k1``, ``k2``: This allows
  the user to use these variables without having to declare them
  manually.
- assignment of integers ``nvr_x1``, ``nvr_x2``, ``nvr_y``, ``nr_k1``,
  ``nr_k2`` ``no_x1``, ``no_x2``, ``no_y``, ``ng_1``: These constants
  are convenient in accessing the attributes of the ``xbe``. For
  example, ``X.val_vr[nvr_x1]`` gives the current value of ``x1`` for
  this element.

It is now easy to see the following points about ``sum_2.xbe``:

- If an explicit method is being used, the template
  evaluates ``y`` in terms of ``x1`` and ``x2``.
- If an implicit method is being used, the template supplies
  information about the equation it satisfies, viz.,

  .. math::

     g_1 \equiv y - (k_1x_1 + k_2x_2) = 0.

  If the main program is requesting the function value,
  :math:`g_1(x_1,x_2,y)` is
  evaluated; if it is requesting the derivatives, then
  :math:`\displaystyle\frac{\partial g_1}{\partial x_1}`,
  :math:`\displaystyle\frac{\partial g_1}{\partial x_2}`,
  :math:`\displaystyle\frac{\partial g_1}{\partial y}`
  are evaluated.
- If the program is requesting assignment of output parameters, the
  parameters listed in the ``outparms`` statement of ``sum_2.xbe``
  are assigned.

``integrator.xbe``
------------------

Next, we consider an element of type ``integrate``, viz., the
integrator, which satisfies

.. math::
   :label: eq_integrator

   y = k\,\int x\,dt,

where ``x`` and ``y`` are the input and
output variables, respectively, and ``k`` is a real parameter.
Since GSEIM expects the equations to be written
in the general form

.. math::

   \displaystyle\frac{dy}{dt} = f(x_1,x_2,..),

we rewrite Eq. :eq:`eq_integrator` as

.. math::

   \displaystyle\frac{dy}{dt} = \,k\,x.

For ``integrate`` type elements, we also need to specify
the initial or *start-up* value of the state variable(s).
For the integrator, we will denote that by :math:`y_0`.

The integrator template is shown below.

.. code-block:: text
   :linenos:

   xbe name=integrator integrate=yes
   # y = k int (x dt)
   Jacobian: constant
   input_vars: x
   output_vars: y
   aux_vars:
   iparms:
   sparms:
   rparms: k=1
   stparms: y_st=0
   igparms: y_ig=0
   outparms: x y
   n_f= 1
   f_1: d_dt(y)
   n_g= 1
   g_1: x
   C:
   prototypes:
   variables:
   source:
      if (G.flags[G.i_one_time_parms]) {
        return;
      }
      if (G.flags[G.i_init_guess]) {
        X.val_vr [nvr_y] = X.igprm[nig_y_ig];
        return;
      }
      if (G.flags[G.i_startup]) {
        if (G.flags[G.i_explicit]) {
          X.val_vr[nvr_y] = X.stprm[nst_y_st];
        } else if (G.flags[G.i_implicit]) {
          X.h[nf_1] = X.val_vr[nvr_y] - X.stprm[nst_y_st];
        }
        return;
      }
      if (G.flags[G.i_outvar]) {
        X.outprm[no_x] = X.val_vr[nvr_x];
        X.outprm[no_y] = X.val_vr[nvr_y];
        return;
      }
      if (G.flags[G.i_trns]) {
        if (G.flags[G.i_explicit]) {
          if (G.flags[G.i_alg_loop]) {
            X.h[nf_1] = X.val_vr[nvr_y] - X.val_vr_u[nvr_y];
          } else {
            k = X.rprm[nr_k];
            x = X.val_vr[nvr_x];
            X.f[nf_1] = k*x;
          }
        } else if (G.flags[G.i_implicit]) {
          k = X.rprm[nr_k];
          x = X.val_vr[nvr_x];
          if (G.flags[G.i_function]) {
            X.g[ng_1] = k*x;
          }
          if (G.flags[G.i_jacobian]) {
            J.dgdvr[ng_1][nvr_x] = k;
          }
        }
        return;
      }
   endC
   endxbe

The structure of ``integrator.xbe`` is similar to that of
:ref:`sum_2.xbe <sum_2>`; here, we will only point out the
new features.

The start-up parameter ``y_st`` corresponds to :math:`y_0`
mentioned above. The fact that time derivative of ``y`` is
involved in the element equation is indicated by the
``f_1`` statement.

In the C++ part of the template,
we have different sections for start-up and transient simulation.
In the start-up section, the equation :math:`y = y_0` is handled.
In the transient section, if the method is explicit, only the
function :math:`f_1 = k\,x` is evaluated; if it is implicit,
the function :math:`g_1 = k\,x` as well as its derivative
:math:`\displaystyle\frac{\partial g_1}{\partial x}` are computed.

``indmc1.xbe``
------------------

We now look at a more complex element of type ``integrate``,
viz., ``indmc1.xbe``, which implements the induction machine model
given by,

.. math::
   :label: eq_indmc_1

   \displaystyle\frac{d{\psi}_{ds}}{dt} = v_{ds}-r_si_{ds},

.. math::
   :label: eq_indmc_2

   \displaystyle\frac{d{\psi}_{qs}}{dt} = v_{qs}-r_si_{qs},

.. math::
   :label: eq_indmc_3

   \displaystyle\frac{d{\psi}_{dr}}{dt} = -\,\frac{P}{2}\,\omega_ {rm}\psi _{qr}-r_ri_{dr},

.. math::
   :label: eq_indmc_4

   \displaystyle\frac{d{\psi}_{qr}}{dt} =  \frac{P}{2}\,\omega_ {rm}\psi _{dr}-r_ri_{qr},

.. math::
   :label: eq_indmc_5

   \displaystyle\frac{d{\omega}_{rm}}{dt} = \frac{1}{J}\,(T_{em}-T_L),

where

.. math::
   :label: eq_indmc1_1

   i_{ds} = \frac{L_r}{L_m L_e}\,\psi _{ds} - \frac{1}{L_e}\,\psi _{dr},

.. math::
   :label: eq_indmc1_2

   i_{qs} = \frac{L_r}{L_m L_e}\,\psi _{qs} - \frac{1}{L_e}\,\psi _{qr},

.. math::
   :label: eq_indmc1_3

   i_{dr} = \frac{1}{L_m}\,\psi _{ds} - \left(\frac{L_{ls}}{L_m}+1\right)~i _{ds},

.. math::
   :label: eq_indmc1_4

   i_{qr} = \frac{1}{L_m}\,\psi _{qs} - \left(\frac{L_{ls}}{L_m}+1\right)~i _{qs},

.. math::
   :label: eq_indmc1_5

   T_{em} = \frac{3}{4}\, P L_m\,(i_{qs}i_{dr} - i_{ds}i_{qr}),

with

.. math::
   :label: eq_indmc2_1

   L_e = \displaystyle\frac{L_sL_r}{L_m}-L_m`,

.. math::
   :label: eq_indmc2_2

   L_s = L_{ls}+L_m`,

.. math::
   :label: eq_indmc2_3

   L_r = L_{lr}+L_m`.

Since the ``indmc.xbe`` template is rather long, we will split
it into several pieces for the purpose of discussion. The complete
template can be found in ``~/gseim_gui/gseim/xbe/``.

Here is the overall template, without the C++ part:

.. _indmc1:

.. code-block:: text
   :linenos:

   xbe name=indmc1 integrate=yes
   # induction motor model
   Jacobian: variable
   input_vars: vqs vds tl
   output_vars: wrm
   aux_vars:
   +  psids psidr psiqs psiqr
   iparms:
   +  poles=4
   sparms:
   rparms:
   +  rs=0.435
   +  lls=0.002
   +  lm=0.0693
   +  llr=0.002
   +  rr=0.816
   +  j=0.089
   +  ls=0
   +  lr=0
   +  le=0
   +  l1=0
   +  l2=0
   +  l3=0
   +  x1=0
   +  x2=0
   stparms:
   +  psids0=0
   +  psiqs0=0
   +  psidr0=0
   +  psiqr0=0
   +  wrm0=0
   igparms:
   outparms:
   +  wrm
   +  tem
   +  vds
   +  vqs
   +  ia
   +  ib
   +  ic
   n_f= 5
   f_1: d_dt(psids)
   f_2: d_dt(psiqs)
   f_3: d_dt(psidr)
   f_4: d_dt(psiqr)
   f_5: d_dt(wrm)
   n_g= 5
   g_1: vds psids psidr
   g_2: vqs psiqs psiqr
   g_3: wrm psiqr psids psidr
   g_4: wrm psidr psiqs psiqr
   g_5: tl psids psidr psiqs psiqr
   C:
   ....
   ....
   endC
   endxbe

The input variables are
``vqs``, ``vds``, ``tl``, and the output variable is ``wrm``.
In addition, it has internal (auxiliary) variables
``psi_ds``,
``psi_dr``,
``psi_qs``,
``psi_qr``
which are involved in the model equations.
The statements ``f_1``, ``f_2``, etc.
are used to inform the simulator which derivative is involved in that equation.
The statements ``g_1``, ``g_2``, etc.
are used to indicate which variables are involved in the right-hand
side of the corresponding equation.

In the induction machine equations,
there are some *one-time* calculations, e.g., calculation of :math:`L_e`
(Eq. :eq:`eq_indmc2_1`),
which are not required to be performed in every time step. GSEIM provides
a flag for this purpose, as seen from in the following C++ section of the
template.

.. code-block:: text
   :linenos:

   if (G.flags[G.i_one_time_parms]) {
     k4 = 0.5*(sqrt(3.0));

     lls = X.rprm[nr_lls];
     lm  = X.rprm[nr_lm ];
     llr = X.rprm[nr_llr];

     ls = lls + lm;
     lr = llr + lm;
     le = (ls*lr/lm) - lm;
     l1 = lr/(lm*le);
     l2 = 1.0 + (lls/lm);
     l3 = lls/lm;

     X.rprm[nr_ls] = ls;
     X.rprm[nr_lr] = lr;
     X.rprm[nr_le] = le;
     X.rprm[nr_l1] = l1;
     X.rprm[nr_l2] = l2;
     X.rprm[nr_l3] = l3;

     poles = X.iprm[ni_poles];
     p = (double)(poles);
     x1 = 0.75*p*lm;
     x2 = 0.5*p;

     X.rprm[nr_x1] = x1;
     X.rprm[nr_x2] = x2;

     return;
   }

When this flag is
set by the main program, the template computes :math:`L_e` and other one-time
parameters, and saves them in the ``X.rprm`` vector. These
parameters need not be computed again during simulation.

Next, we look at the function assignment sections of ``indmc1.xbe``:

.. code-block:: text
   :linenos:

   if (G.flags[G.i_trns]) {
     if (G.flags[G.i_explicit]) {
       if (G.flags[G.i_alg_loop]) {
         X.h[nf_1] = X.val_aux[na_psids] - X.val_aux_u[na_psids];
         X.h[nf_2] = X.val_aux[na_psiqs] - X.val_aux_u[na_psiqs];
         X.h[nf_3] = X.val_aux[na_psidr] - X.val_aux_u[na_psidr];
         X.h[nf_4] = X.val_aux[na_psiqr] - X.val_aux_u[na_psiqr];
         X.h[nf_5] = X.val_vr [nvr_wrm ] - X.val_vr_u [nvr_wrm ];
       } else {
         rs  = X.rprm[nr_rs ];
         lls = X.rprm[nr_lls];
         lm  = X.rprm[nr_lm ];
         rr  = X.rprm[nr_rr ];
         j   = X.rprm[nr_j  ];
         le  = X.rprm[nr_le ];
         l1  = X.rprm[nr_l1 ];
         l2  = X.rprm[nr_l2 ];
         l3  = X.rprm[nr_l3 ];
         x1  = X.rprm[nr_x1 ];
         x2  = X.rprm[nr_x2 ];

         vqs = X.val_vr[nvr_vqs];
         vds = X.val_vr[nvr_vds];
         wrm = X.val_vr[nvr_wrm];
         tl  = X.val_vr[nvr_tl ];

         psids = X.val_aux[na_psids];
         psidr = X.val_aux[na_psidr];
         psiqs = X.val_aux[na_psiqs];
         psiqr = X.val_aux[na_psiqr];

         ids = (l1*psids) - (psidr/le);
         iqs = (l1*psiqs) - (psiqr/le);

         idr = (psids/lm) - (l2*ids);
         iqr = (psiqs/lm) - (l2*iqs);

         tem0 = x1*(iqs*idr-ids*iqr);
         wr   = x2*wrm;

         X.f[nf_1] = vds-rs*ids;
         X.f[nf_2] = vqs-rs*iqs;
         X.f[nf_3] = (-wr)*psiqr-rr*idr;
         X.f[nf_4] = ( wr)*psidr-rr*iqr;
         X.f[nf_5] = (tem0-tl)/j;
       }
     } else {
       rs  = X.rprm[nr_rs ];
       lls = X.rprm[nr_lls];
       lm  = X.rprm[nr_lm ];
       rr  = X.rprm[nr_rr ];
       j   = X.rprm[nr_j  ];
       le  = X.rprm[nr_le ];
       l1  = X.rprm[nr_l1 ];
       l2  = X.rprm[nr_l2 ];
       l3  = X.rprm[nr_l3 ];
       x1  = X.rprm[nr_x1 ];
       x2  = X.rprm[nr_x2 ];

       vqs = X.val_vr[nvr_vqs];
       vds = X.val_vr[nvr_vds];
       wrm = X.val_vr[nvr_wrm];
       tl  = X.val_vr[nvr_tl ];

       psids = X.val_aux[na_psids];
       psidr = X.val_aux[na_psidr];
       psiqs = X.val_aux[na_psiqs];
       psiqr = X.val_aux[na_psiqr];

       if (G.flags[G.i_function] || G.flags[G.i_jacobian]) {
         ids = (l1*psids) - (psidr/le);
         iqs = (l1*psiqs) - (psiqr/le);
         idr = (psids/lm) - (l2*ids);
         iqr = (psiqs/lm) - (l2*iqs);
         tem0 = x1*(iqs*idr-ids*iqr);
         wr = x2*wrm;

         if (G.flags[G.i_function]) {
           X.g[ng_1] = vds-rs*ids;
           X.g[ng_2] = vqs-rs*iqs;
           X.g[ng_3] = (-wr)*psiqr-rr*idr;
           X.g[ng_4] = ( wr)*psidr-rr*iqr;
           X.g[ng_5] = (tem0-tl)/j;
         }
       }
       if (G.flags[G.i_jacobian]) {
         ids_psids = l1;
         ids_psidr = -1.0/le;

         iqs_psiqs = l1;
         iqs_psiqr = -1.0/le;

         idr_psids = (1.0/lm) - (l2*ids_psids);
         idr_psidr =          - (l2*ids_psidr);

         iqr_psiqs = (1.0/lm) - (l2*iqs_psiqs);
         iqr_psiqr =          - (l2*iqs_psiqr);

         tem0_iqs =  x1*idr;
         tem0_idr =  x1*iqs;
         tem0_ids = -x1*iqr;
         tem0_iqr = -x1*ids;

         tem0_psids =
           tem0_idr*idr_psids +
           tem0_ids*ids_psids;

         tem0_psidr =
           tem0_idr*idr_psidr +
           tem0_ids*ids_psidr;

         tem0_psiqs =
           tem0_iqs*iqs_psiqs +
           tem0_iqr*iqr_psiqs;

         tem0_psiqr =
           tem0_iqs*iqs_psiqr +
           tem0_iqr*iqr_psiqr;

         wr_wrm = x2;

         J.dgdvr[ng_1][nvr_vds] = 1.0;
         J.dgdaux[ng_1][na_psids] = -rs*ids_psids;
         J.dgdaux[ng_1][na_psidr] = -rs*ids_psidr;

         J.dgdvr[ng_2][nvr_vqs] = 1.0;
         J.dgdaux[ng_2][na_psiqs] = -rs*iqs_psiqs;
         J.dgdaux[ng_2][na_psiqr] = -rs*iqs_psiqr;

         J.dgdvr[ng_3][nvr_wrm] = (-wr_wrm)*psiqr;
         J.dgdaux[ng_3][na_psiqr] = (-wr);
         J.dgdaux[ng_3][na_psids] = -rr*idr_psids;
         J.dgdaux[ng_3][na_psidr] = -rr*idr_psidr;

         J.dgdvr[ng_4][nvr_wrm] = (wr_wrm)*psidr;
         J.dgdaux[ng_4][na_psidr] = wr;
         J.dgdaux[ng_4][na_psiqs] = -rr*iqr_psiqs;
         J.dgdaux[ng_4][na_psiqr] = -rr*iqr_psiqr;

         J.dgdvr[ng_5][nvr_tl] = -1.0/j;
         J.dgdaux[ng_5][na_psids] = tem0_psids/j;
         J.dgdaux[ng_5][na_psidr] = tem0_psidr/j;
         J.dgdaux[ng_5][na_psiqs] = tem0_psiqs/j;
         J.dgdaux[ng_5][na_psiqr] = tem0_psiqr/j;
       }
     }
     return;
   }

In the explicit case,
the function :math:`f_1` (i.e.,
``X.f[nf_1]``) is computed as per the right-hand side of
Eq. :eq:`eq_indmc_1`, and so on.
In the implicit case,
the function :math:`g_1` is computed in a similar manner. Note that,
in this case, the derivatives of :math:`g_1` with respect to each of the variables involved
in that equation are also computed.

The output parameter computation section of the template is
given below:

.. code-block:: text
   :linenos:

   if (G.flags[G.i_outvar]) {
     X.outprm[no_wrm] = X.val_vr[nvr_wrm];
     X.outprm[no_vds] = X.val_vr[nvr_vds];
     X.outprm[no_vqs] = X.val_vr[nvr_vqs];

     psids = X.val_aux[na_psids];
     psidr = X.val_aux[na_psidr];
     psiqs = X.val_aux[na_psiqs];
     psiqr = X.val_aux[na_psiqr];

     le  = X.rprm[nr_le];
     lm  = X.rprm[nr_lm];
     l1  = X.rprm[nr_l1];
     l2  = X.rprm[nr_l2];

     ids = (l1*psids) - (psidr/le);
     iqs = (l1*psiqs) - (psiqr/le);
     idr = (psids/lm) - (l2*ids);
     iqr = (psiqs/lm) - (l2*iqs);

     X.outprm[no_ia] = iqs;
     X.outprm[no_ib] = -0.5*iqs-k4*ids;
     X.outprm[no_ic] = -0.5*iqs+k4*ids;

     x1 = X.rprm[nr_x1];
     tem0 = x1*(iqs*idr-ids*iqr);
     X.outprm[no_tem] = tem0;

     return;
   }

For :ref:`indmc1.xbe <indmc1>`, the output parameter are
``wrm``, ``tem``, ``vds``, ``vqs``, ``ia``, ``ib``, ``ic``.
To assign the current value of the *variable* ``wrm`` to the
*output parameter* ``wrm``, we need to assign
``X.outprm[no_wrm]``, and so on. Note that
``ia``, ``ib``, ``ic``, ``tem`` are not readily available (they
are not input, output, or auxiliary variables of this template),
and therefore need to be computed and then assigned.
