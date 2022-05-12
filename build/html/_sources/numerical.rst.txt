
.. _numerical:

##########################
Numerical methods for ODEs
##########################

In this section, we will look at numerical methods, in particular,
those employed in GSEIM. Although this section is not strictly
required for using GSEIM, it would help the user to develop a
better understanding of GSEIM (and to some extent other simulators
with similar capabilities). Most of the material presented in this
section is taken directly from the
`SEQUEL manual <https://www.ee.iitb.ac.in/~sequel/>`_. The reader
may find it useful to check out the references listed there.

.. _explicit:

================
Explicit methods
================

Consider the ODE,

.. math::
   :label: neq_ode_1

   \displaystyle\frac{dx}{dt} = f(t,x),~~~x(t_0) = x_0.

In order to solve the ODE numerically, we **discretise** the time
axis from
:math:`t_{\mathrm{start}}` to
:math:`t_{\mathrm{end}}`
as shown below, and then replace the original ODE with an approximate
equation that depends on the numerical method being employed.

.. _discretise:

.. figure:: ode1.png
   :width: 500
   :alt: time discretisation

   Time discretisation

We are interested in getting a numerical solution i.e., the *discrete*
values :math:`x_1`, :math:`x_2`, :math:`\cdots`, corresponding to
:math:`t_1`, :math:`t_2`, etc. Throughout this section, we will denote
the exact solution by :math:`x(t)`; e.g., :math:`x(t_1)` means the exact solution
at :math:`t_1`. On the other hand, we will denote the *numerical* solution at
:math:`t_1` by :math:`x_1`.

Forward Euler method
====================

At :math:`t = t_0`, we start with
:math:`x = x_0` (the initial condition).
From Eq. :eq:`neq_ode_1`, we can compute the slope of :math:`x(t)`
at :math:`t_0` which is simply :math:`f(t_0,x_0)`. In the Forward Euler (FE)
scheme, we make the approximation that this slope applies to the
entire interval :math:`(t_0,t_1)`, i.e., from the current time point to
the next time point. With this assumption, :math:`x(t_1)` is given by
:math:`x_1 = x_0+(t_1-t_0)\times f(t_0,x_0)`, as shown in the figure below.

.. image:: ode2.png
  :width: 330
  :alt: forward euler method

Similarly, from :math:`x_1`, we can get :math:`x_2`, and
so on. In general, we have

.. math::
   :label: neq_ode_2

    x_{n+1} = x_n + \Delta t_n f(t_n,x_n),

where :math:`\Delta t_n = t_{n+1}-t_n`.
Let us apply the FE method to the ODE,

.. math::
   :label: neq_ode_3

   \displaystyle\frac{dx}{dt} = a\left(\sin \omega t -x\right),~~x(0)=0,

which has the analytical (exact) solution,

.. math::
   :label: neq_ode_4

   x(t) =
   \displaystyle\frac{a\omega}{a^2+\omega ^2}\left(e^{-at}-\cos \omega t\right) +
   \displaystyle\frac{a^2}{a^2+\omega ^2}\,\sin \omega t.

The following C program can be used to obtain the numerical solution
of Eq. :eq:`neq_ode_3` with the FE method.

.. code-block:: text
   :linenos:

   #include<stdio.h>
   #include<math.h>

   int main()
   {
     double t,t_start,t_end,h,x,x0,f;
     double a,w;
     FILE *fp;

     x0=0.0; t_start=0.0; t_end=5.0; h=0.05;
     a=1.0; w=5.0;

     fp=fopen("fe1.dat","w");

     t=t_start; x=x0;
     fprintf(fp,"%13.6e %13.6e\n",t,x);

     while (t <= t_end) {
       f = a*(sin(w*t)-x);
       x = x + h*f;
       t = t + h;
       fprintf(fp,"%13.6e %13.6e\n",t,x);
     }
     fclose(fp);
   }

The numerical solution along with the analytical (exact) solution given by
Eq. :eq:neq_ode_4 are shown below.

.. image:: ode3a.png
  :width: 400
  :alt: FE solution

Note that the numerical solution appears to be continuous; however, in reality,
it consists of discrete points which are generally connected with line segments
serving as a "guide to the eye." We notice some difference between
the two, but it can be made smaller by using a smaller step size :math:`h`.
(Readers new to numerical analysis should try this out by running the program
with a smaller value of :math:`h`, e.g., :math:`h = 0.02`. In these matters,
there is no substitute for hands-on experience.)

In general, the accuracy of a numerical method for solving ODEs is described by
the **order** of the method which in turn depends on the **Local Truncation Error**
(LTE) for that method. The LTE is a measure of the *local* error (i.e., error
made in a single time step) and is defined as [Shampine, 1994],

.. math::
   :label: neq_ode_5

   {\textrm{LTE}} = x(t_{n+1})-x_{n+1},

where :math:`x(t_{n+1})` is the exact solution at :math:`t_{n+1}`, and
:math:`u_{n+1}` is the solution obtained by the numerical method,
starting with the exact solution :math:`x(t_n)` at :math:`t = t_n`.
If our numerical method was perfect, :math:`x_{n+1}` would be the same as
:math:`x(t_{n+1})`, and the LTE would be zero.

Let us look at the LTE of the FE method for the test equation,

.. math::
   :label: neq_ode_6

   \displaystyle\frac{dx}{dt} = -\lambda x,~~~x(0)=1,~~\lambda > 0,

with the exact solution

.. math::
   :label: neq_ode_7

   x = e^{-\lambda t}.

To compute the LTE, we take a specific :math:`t_n`, say,  
:math:`t_n = t_0 = 0`, and compute
:math:`x_{n+1}` (i.e., :math:`x_1`) by performing one step of the FE method,
starting with :math:`x_0 = 1`.
The exact solution at :math:`t = t_{n+1} = h` is
:math:`x(h) = e^{-\lambda h}`. The LTE is the difference between
:math:`x_1` and :math:`x(h)`.

The LTE (magnitude) as a function of time step :math:`h` is shown in the
following figure. As :math:`h` is reduced by a factor of 10 (from
:math:`10^{-1}` to :math:`10^{-2}`, for example), the LTE goes down by
two orders of magnitude. In other words, the LTE varies as :math:`h^2`,
and therefore the FE method is said to be of order 1. In general, if
the :math:`{\textrm{LTE}}~\sim h^{k+1}` for a numerical method, then
it is said to be of order :math:`k`.

.. _lteferk4:

.. figure:: ode4a.png
   :width: 400

   LTE for FE and RK4

Runge-Kutta method of order 4
=============================

The Runge-Kutta method of order 4 (the *classic* form) is given by

.. math::
   :label: neq_ode_8

   \begin{align}
    f_0 &= f(t_n,x_n), \\
    f_1 &= f\left(t_n+\displaystyle\frac{h}{2},x_n+\displaystyle\frac{h}{2}\,f_0\right), \\
    f_2 &= f\left(t_n+\displaystyle\frac{h}{2},x_n+\displaystyle\frac{h}{2}\,f_1\right), \\
    f_3 &= f\left(t_n+h,x_n+h\,f_2\right), \\
    x_{n+1} &=
      x_n + h
     \left(
      \displaystyle\frac{1}{6}\,f_0 +
      \displaystyle\frac{1}{3}\,f_1 +
      \displaystyle\frac{1}{3}\,f_2 +
      \displaystyle\frac{1}{6}\,f_3
     \right).
   \end{align}

The order of the RK4 method can be made out from :ref:`lteferk4`:
if :math:`h` is reduced by one order of magnitude, the LTE goes down by
five orders of magnitude (2.5 divisions, with each division
corresponding to two decades), and therefore the order is four.

Since the RK4 method is of a higher order, we expect it to be more
accurate than the FE method. Let us check that by comparing the
numerical solutions obtained with the two methods for the ODE
given by :eq:`neq_ode_3`. The following program can be used
for the RK4 method.

.. code-block:: text
   :linenos:

   #include<stdio.h>
   #include<math.h>

   int main()
   {
     double t,t_start,t_end,h,x,x0;
     double f0,f1,f2,f3;
     double a,w;
     FILE *fp;

     x0=0.0; t_start=0.0; t_end=5.0; h=0.05;
     a=1.0; w=5.0;

     fp=fopen("rk4.dat","w");

     t=t_start; x=x0;
     fprintf(fp,"%13.6e %13.6e\n",t,x);

     while (t <= t_end) {
       f0 = a*(sin(w*t)-x);
       f1 = a*(sin(w*(t+0.5*h))-(x+0.5*h*f0));
       f2 = a*(sin(w*(t+0.5*h))-(x+0.5*h*f1));
       f3 = a*(sin(w*(t+h))-(x+h*f2));

       x = x + (h/6.0)*(f0+f1+f1+f2+f2+f3);
       t = t + h;
       fprintf(fp,"%13.6e %13.6e\n",t,x);
     }
     fclose(fp);
   }

The numerical solutions for :eq:`neq_ode_3` obtained with
the FE and RK4 methods (crosses and squares, respectively)
using a step size of :math:`h = 0.05` are shown in the figure
below. The exact solution is also shown (red curve).
Clearly, the RK4 method is more accurate.

.. image:: ode5a.png
  :width: 600
  :alt: FE and RK4 comparison

Both FE and RK4 are **explicit** methods in the sense that
:math:`x_{n+1}` can be computed simply by evaluating quantities based
on the past values of :math:`x`
(see Eqs. :eq:`neq_ode_2` and :eq:`neq_ode_8`).
The RK4 method is more complicated as it involves computation
of the function values
:math:`f_0`,
:math:`f_1`,
:math:`f_2`,
:math:`f_3`,
but the entire computation can be completed in a step-by-step
manner. For example, the computation of :math:`f_2` requires only
:math:`x_n` and :math:`f_1`, which are already available. The simplicity
of the computation is reflected in the programs we have seen.

.. _system_ode:

System of ODEs
==============

The above methods (and other explicit methods) can be easily extended
to a set of ODEs of the form

.. math::
   :label: neq_ode_9

   \begin{align}
    \displaystyle\frac{dx_1}{dt} &= f_1(t,x_1,x_2,\cdots,x_N), \\
    \displaystyle\frac{dx_2}{dt} &= f_2(t,x_1,x_2,\cdots,x_N), \\
    & ~~~~\vdots \\
    \displaystyle\frac{dx_N}{dt} &= f_N(t,x_1,x_2,\cdots,x_N),
   \end{align}

with the initial conditions at :math:`t = t_0` specified as
:math:`x_1(t_0) = x_1^{(0)}`,
:math:`x_2(t_0) = x_2^{(0)}`, etc.
With vector notation, we can write the above set of equations as

.. math::
   :label: neq_ode_10

   \displaystyle\frac{d {\bf{x}}}{dt} = {\bf{f}}(t,{\bf{x}}),~~~{\bf{x}}(t_0) = {\bf{x}}^{(0)}.

The FE method for this set of ODEs can be written as

.. math::
   :label: neq_ode_11

   {\bf{x}}^{(n+1)} = {\bf{x}}^{(n)} + h\,{\bf{f}}(t_n,{\bf{x}}^{(n)}),

and the RK4 method as

.. math::
   :label: neq_ode_12

   {\bf{f}}_0
   &= {\bf{f}}(t_n,{\bf{x}}^{(n)}),
   \\
   {\bf{f}}_1
   &= {\bf{f}}(t_n+\displaystyle\frac{h}{2},{\bf{x}}^{(n)}+
     \displaystyle\frac{h}{2}\,{\bf{f}}_0),
   \\
   {\bf{f}}_2
   &= {\bf{f}}(t_n+\displaystyle\frac{h}{2},{\bf{x}}^{(n)}+
     \displaystyle\frac{h}{2}\,{\bf{f}}_1),
   \\
   {\bf{f}}_3
   &= {\bf{f}}(t_n+h,{\bf{x}}^{(n)}+h\,{\bf{f}}_2),
   \\
   {\bf{x}}^{(n+1)}
   &= {\bf{x}}^{(n)} + h
    \left(
     \displaystyle\frac{1}{6}\,{\bf{f}}_0 +
     \displaystyle\frac{1}{3}\,{\bf{f}}_1 +
     \displaystyle\frac{1}{3}\,{\bf{f}}_2 +
     \displaystyle\frac{1}{6}\,{\bf{f}}_3
    \right),

where :math:`{\bf{x}}^{(n)}` denotes the solution vector at time :math:`t_n`.
As in the single-equation case, the evaluations can be performed in a step-by-step
manner, enabling a straightforward implementation.

The simplicity of explicit methods makes them very attractive. Furthermore,
the implementation remains easy even if the functions :math:`f_i` are nonlinear.
Let us illustrate this point with an example. Consider the system of
ODEs given by

.. math::
   :label: neq_ode_13

   \displaystyle\frac{dx_1}{dt}
   &= a_1\left(\sin \omega t -x_1\right)^3-a_2\left(x_1-x_2\right),
   \\
   \displaystyle\frac{dx_2}{dt}
   &= a_3\left(x_1-x_2\right),

with the initial condition,
:math:`x_1(0) = 0`,
:math:`x_2(0) = 0`. If we use the FE method to solve the equations, we get

.. math::
   :label: neq_ode_14

   x_1^{(n+1)}
   &=
   x_1^{(n)} +
   h\left[a_1\left(\sin \omega t_n - x_1^{(n)}\right)^3-a_2\left(x_1^{(n)}-x_2^{(n)}\right)\right],
   \\
   x_2^{(n+1)}
   &=
   x_2^{(n)} +
   h\left[a_3\left(x_1^{(n)}-x_2^{(n)}\right)\right].

The following program, which is a straightforward extension of our earlier
FE program, can be used to implement the above equations.

.. code-block:: text
   :linenos:

   #include<stdio.h>
   #include<math.h>

   int main()
   {
      double t,t_start,t_end,h;
      double x1,x2,f1,f2,b1;
      double w,a1,a2,a3,f_hz,pi;
      FILE *fp;

      f_hz = 5.0e3; // frequency = 5 kHz
      pi = acos(-1.0);
      w = 2.0*pi*f_hz;
  
      a1 = 5.0e3;
      a2 = 5.0e3;
      a3 = 5.0e3;

      t_start=0.0;
      t_end=5.0e-3;
      h=0.001e-3; // time step = 0.001 msec

      x1=0.0;
      x2=0.0;

      fp=fopen("fe3.dat","w");

      t=t_start;
      fprintf(fp,"%13.6e %13.6e %13.6e\n",t,x1,x2);

      while (t <= t_end) {
        b1 = sin(w*t)-x1;
        f1 = a1*b1*b1*b1 - a2*(x1-x2);
        f2 = a3*(x1-x2);
        x1 = x1 + h*f1;
        x2 = x2 + h*f2;
        t = t + h;
        fprintf(fp,"%13.6e %13.6e %13.6e\n",t,x1,x2);
      }
      fclose(fp);
   }

As simple as that! The numerical solution is shown below.

.. image:: ode6a.png
  :width: 600
  :alt: FE nonlinear example

.. _ode_adaptive:

Adaptive time step
==================

We have so far considered the time step :math:`h` to be
constant. When the solution has regions of fast
and slow variations, it is much more efficient to
use adaptive (variable) time steps. When the solution
varies rapidly, small time steps are required to capture
the transients accurately; at other times, larger time steps can
be used.  In this way, the total number of time points -- and thereby
the simulation time -- can be made
much smaller than using a small, uniform time step.

In order to implement an adaptive time step scheme,
we need a mechanism to judge the accuracy of the numerical
solution. Ideally, we would like to
use the difference between the analytical solution
and the numerical solution, i.e., :math:`|x(t_n)-x_n|`.
However, since the analytical solution is not available, we
need some other means of checking the accuracy. A commonly
used method is to estimate the local truncation error by
computing :math:`x_{n+1}` with two numerical methods: one method of order
:math:`p` and the other of order :math:`p+1`. Let
:math:`{\textrm{LTE}}^{(p)}` and
:math:`{\textrm{LTE}}^{(p+1)}` denote the
local truncation errors in going from :math:`t_n` to :math:`t_{n+1}` for
the two methods, and let
:math:`x_{n+1}` and :math:`\tilde{x}_{n+1}` be the corresponding
numerical solutions. As we have seen,
:math:`{\textrm{LTE}}^{(p)}` and
:math:`{\textrm{LTE}}^{(p+1)}` are
:math:`O(h^{p+1})` and
:math:`O(h^{p+2})`, respectively. If we assume :math:`x_n` to be equal to
the exact solution :math:`x(t_n)`, we have

.. math::
   :label: neq_ode_adaptive_1

   {\textrm{LTE}}^{(p)} = x(t_{n+1})-x_{n+1} ,

.. math::
   :label: neq_ode_adaptive_2

   {\textrm{LTE}}^{(p+1)} = x(t_{n+1})-\tilde{x}_{n+1} .

Subtracting
Eq. :eq:`neq_ode_adaptive_2` from
Eq. :eq:`neq_ode_adaptive_1`, we get

.. math::
   :label: neq_ode_adaptive_3

   {\textrm{LTE}}^{(p)}- {\textrm{LTE}}^{(p+1)}
   = \tilde{x}_{n+1}- x_{n+1} \,.

Since
:math:`{\textrm{LTE}}^{(p+1)}` is expected to be much smaller than
:math:`{\textrm{LTE}}^{(p)}`, we can ignore it and obtain

.. math::
   :label: neq_ode_adaptive_4

   {\textrm{LTE}}^{(p)} \approx \tilde{x}_{n+1}- x_{n+1} \,.

Having obtained an estimate for the LTE (denoted by
:math:`{\textrm{LTE}}^{\mathrm{est}}`) resulting from
a time step of :math:`h_n = t_{n+1}-t_n`, we can now check if the solution
should be accepted or not. If :math:`{\textrm{LTE}}^{\mathrm{est}}`
is larger than the specified tolerance, we reject the current step and
try a smaller step.
If :math:`{\textrm{LTE}}^{\mathrm{est}}` is
smaller than the tolerance, we accept the solution obtained at
:math:`t_{n+1}` (i.e., :math:`x_{n+1}`). In this case, there is a
possibility that our current time step :math:`h_n` is too conservative,
and we explore whether the next time step (i.e., :math:`h_{n+1} = t_{n+2}-t_{n+1}`)
can be made larger.

A flow chart for implementing adaptive time steps based on the above ideas
is shown in the following figure.

.. image:: ode7.png
  :width: 500
  :alt: adaptive step

The tolerance :math:`\tau` specifies
the maximum value of the LTE per time step (i.e., :math:`{\textrm{LTE}}/h`) that is
acceptable. The method of order :math:`p` is used for actually advancing the
solution, and the method of order :math:`p+1` is used only to compute
:math:`{\textrm{LTE}}^{\mathrm{est}}` using Eq. :eq:`neq_ode_adaptive_4`. 
Since :math:`{\textrm{LTE}}/h` is :math:`O(h^p)`, we can write

.. math::
   :label: neq_ode_adaptive_5

   \displaystyle\frac{{\rm LTE}^{(p)}}{h_n} =
   \displaystyle\frac{|\tilde{x}_{n+1}-x_{n+1}|}{h_n} =
   Kh_n^p.

Next, we compute the time step
(:math:`\equiv \delta \times h_n`) which would result in
an LTE per time step equal to :math:`\tau`, using

.. math::
   :label: neq_ode_adaptive_6

   \tau = K(\delta h_n)^p.

From Eqs. :eq:`neq_ode_adaptive_5` and
:eq:`neq_ode_adaptive_6`, we obtain :math:`\delta` as

.. math::
   :label: neq_ode_adaptive_7

   \delta =
   \left(
    \displaystyle\frac{\tau h_n}{|\tilde{x}_{n+1}-x_{n+1}|}
   \right)^{1/p}.

If the current LTE per time step (in going from :math:`t_n` to :math:`t_{n+1}`) 
is larger than :math:`\tau` (i.e., :math:`\delta < 1`), we reject the
current time step and try a new time step :math:`h_n \leftarrow \delta \times h_n`.
If it is smaller than :math:`\tau`, we accept the current solution (:math:`x_{n+1}`).
In this case, :math:`\delta` is larger than one, and the next time step is
taken to be :math:`h_{n+1} = \delta \times h_n`.

Since drastic changes in the time step are not suitable from the stability
perspective [Shampine, 1994], the value of :math:`\delta` is generally
restricted to :math:`\delta _{\mathrm{min}} \leq \delta \leq \delta _{\mathrm{max}}`,
as shown in the flow chart. Minimum and maximum limits are also imposed on
the step size (:math:`h _{\mathrm{min}}` and
:math:`h _{\mathrm{max}}` in the flow chart).

Different pairs of methods -- of orders :math:`p` and :math:`p+1` -- are available in
the literature for implementing the above scheme.
In the commonly used Runge-Kutta-Fehlberg (RKF45) pair,
which consists of an order-4 method and an order-5 method,
the LTE is estimated from [Burden, 2001]

.. math::
   :label: neq_rkf45_1

   \begin{align}
    x_{n+1}
    &=
    x_n + h_n\left(
    \displaystyle\frac{25}{216}\,f_0
    + \displaystyle\frac{1408}{2565}\,f_2
    + \displaystyle\frac{2197}{4104}\,f_3
    - \displaystyle\frac{1}{5}\,f_4
    \right),
    \\
     \tilde{x}_{n+1}
     &=
     x_n + h_n\left(
    \displaystyle\frac{16}{135}\,f_0
    + \displaystyle\frac{6656}{12825}\,f_2
    + \displaystyle\frac{28561}{56430}\,f_3
    - \displaystyle\frac{9}{50}\,f_4
    + \displaystyle\frac{2}{55}\,f_5
    \right),
   \end{align}

where

.. math::
   :label: neq_rkf45_2

   \begin{align}
    f_0 &=
    f(t_n,x_n),
    \\
    f_1 &=
    \left(
    t_n+\displaystyle\frac{h_n}{4},
    x_n
    +\displaystyle\frac{1}{4}\,f_0
    \right),
    \\
    f_2 &=
    \left(
    t_n+
    \displaystyle\frac{3h_n}{8},
    x_n
    +\displaystyle\frac{3}{32}\,f_0
    +\displaystyle\frac{9}{32}\,f_1
    \right),
    \\
    f_3 &=
    \left(
    t_n+
    \displaystyle\frac{12h_n}{13},
    x_n
    +\displaystyle\frac{1932}{2197}\,f_0
    -\displaystyle\frac{7200}{2197}\,f_1
    +\displaystyle\frac{7296}{2197}\,f_2
    \right),
    \\
    f_4 &=
    \left(
    t_n+h_n,
    x_n
    +\displaystyle\frac{439}{216}\,f_0
    -8\,f_1
    +\displaystyle\frac{3680}{513}\,f_2
    -\displaystyle\frac{845}{4104}\,f_3
    \right),
    \\
    f_5 &=
    \left(
    t_n+
    \displaystyle\frac{h_n}{2},
    x_n
    -\displaystyle\frac{8}{27}\,f_0
    +2\,f_1
    -\displaystyle\frac{3544}{2565}\,f_2
    +\displaystyle\frac{1859}{4104}\,f_3
    -\displaystyle\frac{11}{40}\,f_4
    \right).
   \end{align}

Note that the order-4 part of this method -- the
computation of :math:`x_{n+1}` in Eq. :eq:`neq_rkf45_1` --
is different from the classic RK4 method we have
seen earlier (Eq. :eq:`neq_ode_8`). The order-4 and
order-5 methods in the RKF45 scheme are designed
such that, with little extra computation (over the
order-4 method), we get the order-5 result
(:math:`\tilde{x}_{n+1}` in Eq. :eq:`neq_rkf45_1`).

Let us illustrate the effectiveness of the RKF45 method
with an example. Consider the RC circuit shown below.

.. image:: ode9.png
  :width: 160
  :alt: RC circuit

The behaviour of this circuit is described by the ODE,

.. math::
   :label: neq_rkf_rc_1

   \displaystyle\frac{dv_c}{dt} =
   \displaystyle\frac{1}{\tau}\, (v_s-v_c),

where :math:`\tau = RC`, and :math:`v_s` is a known function
of time. In particular, we will consider :math:`v_s(t)` to be a pulse
going from
:math:`0\,{\textrm{V}}` to
:math:`1\,{\textrm{V}}`
at :math:`t_1 = 0.5\,{\textrm{sec}}`
with a rise time of
:math:`0.05\,{\textrm{sec}}` and from
:math:`1\,{\textrm{V}}` back to
:math:`0\,{\textrm{V}}`
at :math:`t_2 = 2\,{\textrm{sec}}`
with a fall time of
:math:`0.05\,{\textrm{sec}}`.
The solution of Eq. :eq:`neq_rkf_rc_1`
obtained with the RKF45 method with a tolerance
:math:`\tau = 10^{-4}` is shown below.
The other parameters are
:math:`R = 1\,\Omega`,
:math:`C = 1\,{\textrm{F}}`,
:math:`\delta _{\mathrm{min}} = 0.2`,
:math:`\delta _{\mathrm{max}} = 2`,
:math:`h _{\mathrm{min}} = 10^{-5}`,
:math:`h _{\mathrm{max}} = 0.5`.
As we expect, the time steps are small when the solution is changing
rapidly (i.e., near the pulse edges) and large
when it is changing slowly.

.. image:: ode8b.png
  :width: 600
  :alt: RC circuit results

The tolerance :math:`\tau` needs to be chosen carefully. If it is
large, it may not give us a sufficiently accurate solution.
For example, as seen from the following figure, the solution obtained
with :math:`\tau = 10^{-2}` (squares) differs significantly
from that obtained with :math:`\tau = 10^{-4}` (crosses).

.. image:: ode8a.png
  :width: 600
  :alt: RC circuit results

On the other hand, reducing :math:`\tau` beyond
:math:`10^{-4}` does not change the solution any more, as seen
in the following figure showing results for
:math:`\tau = 10^{-4}` (crosses) and
:math:`\tau = 10^{-6}` (blue graph).
Note that we are being somewhat lax about terminology here -- When we say
that "the solution does not change", what we mean is, "I cannot make
out the difference, if any" which is often good enough in practice.
Looking through the microscope for a change in the fifth decimal
place is generally not called for in engineering problems.

.. image:: ode11a.png
  :width: 600
  :alt: RC circuit results

Although there is no perceptible difference in the results when
:math:`\tau` is changed from
:math:`10^{-4}` to
:math:`10^{-6}`, the number of time points does increase.
The following figure shows the total number of time points
(:math:`N_{\mathrm{total}}`) used by the RKF45 method in covering the
time interval from
:math:`t_{\mathrm{start}}` to :math:`t_{\mathrm{end}}` (i.e., 0 to 10 sec)
as a function of :math:`\tau`.

.. image:: ode10a.png
  :width: 350
  :alt: RC circuit results

With a tighter
tolerance (smaller :math:`\tau`), the number of time points to be simulated
goes up and so does the simulation time. For this specific problem,
we would not even notice the difference in the computation time,
but for larger problems (with a large number of variables or a large number
of time points or both), the difference could be substantial.

Sometimes, there is an "aesthetics" issue. What has aesthetics got
to do with science, we may ask. Consider the same RC circuit of
with a sinusoidal voltage source :math:`V_m \sin \omega t`.
The results obtained with
:math:`\tau = 10^{-5}` and
:math:`\tau = 10^{-6}` are shown below.

.. image:: ode12a.png
  :width: 600
  :alt: RC circuit sinusoid

For :math:`\tau = 10^{-5}`, the solution is accurate, i.e.,
the numerical solution agrees closely with the analytical solution.
However, it appears discontinuous because of the small number of
time points.
For :math:`\tau = 10^{-6}`, the RKF45 method forces a larger number
of time points (smaller time steps) in order to meet the tolerance
requirement, and the solution now appears continuous, more in tune with
what we want to see.

.. _explicit_stability:

Stability
=========

Apart from being sufficiently accurate,
a numerical method for solving ODEs must also be
*stable*, i.e., its "global error"
:math:`|x(t_n)-x_n|` -- where
:math:`x(t_n)` and :math:`x_n`
are the exact and numerical solutions,
respectively, at :math:`t = t_n` -- must remain bounded.

Broadly, we can talk of two types of stability.

- Stability for small :math:`h`: Based on our discussion of
  local truncation error, we would expect
  that the accuracy of the numerical solution would
  generally get better if we use smaller step sizes.
  However, this is not true for all numerical methods.
  We can have a method which is accurate to
  a specified order in the local sense but is unstable
  in the global sense (i.e., the numerical solution
  "blows up" as time increases)
  *even if* the time steps are small.
  Such methods are of course of no use in circuit simulation,
  and we will not discuss them here.
- Stability for large :math:`h`: All commonly used numerical methods
  for solving ODEs (including the FE and RK4 methods we have
  seen before) can be expected to work well when the step size
  :math:`h` is sufficiently small. When :math:`h` is increased, we expect
  the solution to be less accurate, but quite apart from that,
  the solution may also become unstable, and that is a serious
  concern. In the following, we will illustrate this point
  with an example.

Consider the RC circuit shown in the figure below.

.. _tworc:

.. figure:: ode16.png
   :width: 600
   :alt: two-RC circuit

   RC circuit with two time constants

This circuit can be described by the ODEs,

.. math::
   :label: neq_tworc_ode_1

   \begin{align}
    \displaystyle\frac{dV_1}{dt} &=
    \displaystyle\frac{1}{R_1C_1}\,(V_s-V_1)-
    \displaystyle\frac{1}{R_2C_1}\,(V_1-V_2),
    \\
    \displaystyle\frac{dV_2}{dt} &=
    \displaystyle\frac{1}{R_2C_2}\,(V_1-V_2).
   \end{align}

With a sinusoidal input,
:math:`V_s = V_m\,\sin \omega t`,
we can use phasors
to estimate :math:`V_1(t)` in steady state.
In particular, let
:math:`R_1 = 1\,{\textrm{k}}\Omega`,
:math:`R_2 = 2\,{\textrm{k}}\Omega`,
:math:`C_1 = 540\,{\textrm{nF}}`,
:math:`C_2 = 1\,{\textrm{mF}}`.
With these component values, and with a frequency
of :math:`{\textrm{50}}\,{\textrm{Hz}}`, we have
:math:`{\bf{Z}}_1 = -j5.9\,{\textrm{k}}\Omega`,
:math:`{\bf{Z}}_2 = -j3.2\,\Omega`.
Since 
:math:`{\bf{Z}}_1` is relatively large, we can replace it with an
open circuit. Similarly, since
:math:`{\bf{Z}}_2` is small, we can replace it with a short circuit.
The approximate solution for
:math:`{\bf{V}}_1` is then simply

.. math::
   :label: neq_tworc_ode_2

   {\bf{V}}_1 \approx {\bf{V}}_s\,\displaystyle\frac{R_2}{R_1+R_2}.

The numerical solution obtained with the RK4 method with a time step
of $h = 1$\,msec is shown below (the light
blue curve). Its amplitude and phase are in agreement with
Eq. :eq:`neq_tworc_ode_2`.

.. image:: ode15a.png
  :width: 550
  :alt: two-RC RK result

Do we expect any changes if :math:`C_1` is changed from
:math:`540\,{\textrm{nF}}` to :math:`535\,{\textrm{nF}}`?
Nothing, really. This change is not going to significantly affect
the value of 
:math:`{\bf{Z}}_1`, and we do not expect the solution to change
noticeably. However, as seen in the figure, the numerical solution
(obtained with the same RK4 method and with the same step size)
is dramatically different. It starts off being similar, but then
takes off toward infinity.

Why does the value of :math:`C_1` make such a
dramatic difference? The answer has to do with stability of
the RK4 method. The following table gives the condition for stability
of a few explicit RK methods when applied to the ODE
:math:`\displaystyle\frac{dx}{dt} = -\,\displaystyle\frac{x}{\tau}`.

+-------------------+----------------------+
|       Method      |  Maximum step size   |
+===================+======================+
| RK-1 (same as FE) |   :math:`2\,\tau`    |
+-------------------+----------------------+
| RK-2              |   :math:`2\,\tau`    |
+-------------------+----------------------+
| RK-3              |   :math:`2.5\,\tau`  |
+-------------------+----------------------+
| RK-4              |   :math:`2.8\,\tau`  |
+-------------------+----------------------+

We see that the RK4 method is unstable when the step size exceeds about
:math:`2.8\, \tau` (:math:`2.785\,\tau` to be more precise). If there
are several time constants governing the set of ODEs being solved,
this limit still holds with :math:`\tau` replaced by the
*smallest* time constant.

Let us see the implications of the above limit in the context of our
RC example described by Eq. :eq:`neq_tworc_ode_1`.
This circuit has two time constants, and they vary with :math:`C_1`
as shown below (with other component values held constant at
their previous values, viz.,
:math:`R_1 = 1\,{\textrm{k}}\Omega`,
:math:`R_2 = 2\,{\textrm{k}}\Omega`,
:math:`C_2 = 1\,{\textrm{mF}}`).

.. image:: ode17a.png
  :width: 550
  :alt: two-RC time constants

The time constant marked :math:`\tau _2` is the smaller of the two,
and it is
:math:`0.356645\,{\textrm{msec}}` and
:math:`0.359978\,{\textrm{msec}}`
for :math:`C_1 = 535\,{\textrm{nF}}`
and :math:`540\,{\textrm{nF}}`, respectively. For each of these,
the maximum time step :math:`h_{\mathrm{max}}` to guarantee
stability of the RK4 method
can be obtained as 
:math:`2.785\,\tau _2`.
For :math:`C_1 = 540\,{\textrm{nF}}`,
:math:`h_{\mathrm{max}}` is 
:math:`1.0025\,{\textrm{msec}}`. The actual time step used for the numerical
solution :math:`(1\,{\textrm{msec}})` is smaller than this value, and the
solution is stable.
For :math:`C_1 = 535\,{\textrm{nF}}`,
:math:`h_{\mathrm{max}}` is 
:math:`0.9933\,{\textrm{msec}}`; the actual time step :math:`(1\,{\textrm{msec}})`
is now larger than this limit, thus leading to instability.

We can handle the instability problem by reducing the RK4 time step.
However, we generally do not have a good idea of the time constants
in a given circuit, and we would then need to carry out a cumbersome
trial-and-error process of choosing a time step, checking if it works,
reducing it if it does not, and so on. In these situations, the
adaptive (auto) time step methods --
such as the RKF45 method discussed in the
:ref:`Adaptive time step <ode_adaptive>` section --
can be used to advantage. When a given time step leads to
instability, the LTE also goes up, and the adaptive step method automatically
reduces the time step suitably *without* any intervention from the user. This is
indeed an attractive choice. Let us illustrate it with an example,
again the :ref:`tworc` but with different component values, viz.,
:math:`R_1 = 1\,{\textrm{k}}\Omega`,
:math:`R_2 = 1\,{\textrm{k}}\Omega`,
:math:`C_1 = 1\,\mu{\textrm{F}}`,
:math:`f = 1\,{\textrm{kHz}}`. As :math:`C_2` is varied, the time
constants change (as seen earlier). When the RKF45 method is used to solve
the ODEs (Eq. :eq:`neq_tworc_ode_1`), the time step is automatically
adjusted to meet the specified tolerance requirement,
and a stable solution is obtained. As :math:`C_2` is made smaller,
the smaller of the two time constants becomes
smaller, and the step sizes employed by the RKF45 algorithms also become
smaller, as shown in the following figure.
(The algorithmic parameters were
:math:`\tau = 10^{-3}`,
:math:`\delta _{\mathrm{min}} = 0.2`,
:math:`\delta _{\mathrm{max}} = 2`,
:math:`h _{\mathrm{min}} = 10^{-5}`,
:math:`h _{\mathrm{max}} = 0.5`.)

.. image:: ode13a.png
  :width: 600
  :alt: two-RC time step

Sounds good, but there is a flip side.
Take for example :math:`C_2 = 10^{-10}\,{\textrm{F}}`. The impedance :math:`{\bf{Z}}_2`
is then :math:`-j64\,{\textrm{M}}\Omega` (at :math:`f = 1\,{\textrm{kHz}}`), and it
is an open circuit for all practical purposes. The circuit reduces to a series
combination of :math:`R_1` and :math:`C_1`, and the solution is independent of
:math:`C_2` (as long as it is small enough). We now have an unfortunate
situation in which :math:`C_2` has no effect on the solution, yet it
forces small time steps because of stability considerations.

The above situation is an example of "stiff" circuits in which the time constants are
vastly different. We are often not interested in tracking the solution of a stiff
circuit on the scale of the smallest time constant, but because of the stability
constraints imposed by the numerical method, we are forced to use small time steps.
This is a drawback of explicit methods since, like the RK4 method, they
are *conditionally* stable.

.. _flowgraphs:

Application to flow graphs
==========================

In this section, we describe how GSEIM handles a flow graph when an explicit
numerical scheme (such as forwar Euler, RK4, RKF45) is used. We will consider
a simple flow graph (see figure) for the purpose of illustration. The signals
have been named as
:math:`x_1`,
:math:`x_2`, etc. and the blocks as
A, B, etc. The flow graph corresponds to the ODE:
:math:`\displaystyle\frac{dx_6}{dt} = k_1x_1 + k_2x_2`,
where
:math:`x_1 = A_1 \sin \omega _1 t`,
:math:`x_2 = A_2 \sin \omega _2 t` (for example).

.. image:: flowgraph1.png
  :width: 300
  :alt: flow graph example 1

The flow graph has the following elements (blocks):

- signal sources (A, B)
- scalar multipliers (C, D)
- summer (E)
- integrator (F)

The integrator equation is
:math:`x_6 = \displaystyle\int x_5dt`, or equivalently,
:math:`\displaystyle\frac{dx_6}{dt} = x_5`. For simplicity,
we will use the forward Euler method to handle the time
derivative involved in the integrator equation. The remaining
elements can be described by algebraic equations, not involving
time derivatives. Denoting the numerical solution for :math:`x_1` at
:math:`t_n` as :math:`x_1^n`, etc., we have the following equations
for the blocks.

- Block **A**: :math:`x_1^{n+1} =  A_1 \,\sin \omega _1 t_{n+1}`
- Block **B**: :math:`x_2^{n+1} =  A_2 \,\sin \omega _2 t_{n+1}`
- Block **C**: :math:`x_3^{n+1} =  k_1 \,x_1^{n+1}`
- Block **D**: :math:`x_4^{n+1} =  k_2 \,x_2^{n+1}`
- Block **E**: :math:`x_5^{n+1} = x_3^{n+1} + x_4^{n+1}`
- Block **F**: :math:`x_6^{n+1} = x_6^n + \Delta t \times x_5^n`

Simple enough. The question is: In what order should we visit these
equations? The order obviously depends on how the elements are
connected in the flow graph. For example, it makes sense to evaluate
the summer output
:math:`x_5^{n+1}`
only after its inputs
:math:`x_3^{n+1}`
:math:`x_4^{n+1}`
have been updated.

The following observations would help in deciding on the order in
which the blocks should be processed.

- The integrator equation involves only the *past* values on the
  right hand side; it can be evaluated independently of the other blocks,
  i.e., we can process the integrator first and then the remaining
  blocks, keeping in mind the flow graph connections.
- Source elements do no have inputs. We can therefore process them
  independently of the other blocks.
- The remaining blocks should be processed in an order which ensures
  that the inputs of a given block are updated before updating its
  outputs.

Using the above considerations, we can come up with the following
options for processing the blocks:

- F :math:`\rightarrow` A :math:`\rightarrow` B :math:`\rightarrow` C :math:`\rightarrow`
  D :math:`\rightarrow` E
- F :math:`\rightarrow` A :math:`\rightarrow` B :math:`\rightarrow` D :math:`\rightarrow`
  C :math:`\rightarrow` E
- F :math:`\rightarrow` B :math:`\rightarrow` A :math:`\rightarrow` C :math:`\rightarrow`
  D :math:`\rightarrow` E
- F :math:`\rightarrow` B :math:`\rightarrow` A :math:`\rightarrow` D :math:`\rightarrow`
  C :math:`\rightarrow` E
- F :math:`\rightarrow` A :math:`\rightarrow` C :math:`\rightarrow` B :math:`\rightarrow`
  D :math:`\rightarrow` E

Note that all of these options are equivalent; the simulator can pick any
of them and use it in each time step.

With this background, we can also understand why 
`Simulink <https://in.mathworks.com/products/simulink.html>`_
processes blocks in a certain order. For example, here is the
Simulink block diagram for the same ODE, i.e.,
:math:`\displaystyle\frac{dx_6}{dt} = k_1x_1 + k_2x_2`.

.. image:: order_sim_1.png
  :width: 500
  :alt: simulink example 1

Simulink uses red labels (see figure) to indicate the order in which
the blocks are processed. For example, for the integrator block, the
label is ``0:0``, where the number after the colon indicates that it
is the first block to be processed. The second block to be processed
is the scope, but we can ignore it since it is used only for plotting.
After that, the top source is processed, and so on. The reader can verify
that the order corresponds to our last option, viz.,
F :math:`\rightarrow` A :math:`\rightarrow` C :math:`\rightarrow`
B :math:`\rightarrow` D :math:`\rightarrow` E.

A slightly more complex Simulink block diagram along with the ordering
annotations is shown below. It is a good exercise to verify that it is
consistent with our remarks.

.. image:: order_sim_2.png
  :width: 650
  :alt: simulink example 2

An important point to note is the *simplicity*
of the simulation process when an explicit method is used.
Each of the above steps is straightforward -- almost
trivial -- to execute. In fact, even a nonlinear operation
(such as :math:`x_6 = x_2\times x_4` in the block marked ``0:5``)
requires no special consideration; it is just another
*evaluation*. In this respect, more complicated explicit
methods, such as the fourth-order Runge-Kutta method or the RKF45
method, are no different. They all involve *evaluation*
of a left-hand side using *known* numbers on the right-hand side.
This is in sharp contrast with
:ref:`implicit methods <implicit>`, as we shall see.

.. _alg_loops:

Algebraic loops
===============

Consider the feedback loop shown in the figure.

.. image:: order_sim_3.png
  :width: 250
  :alt: algebraic loop example

We can write the following equation for this system:

.. math::
   :label: neq_alg_loop_1

   x_3 = k_1\,x_2 = k_1\,(x_1-x_4) = k_1\,(x_1-k_2\,x_3)\,,

giving

.. math::
   :label: neq_alg_loop_2

   x_3 = \displaystyle\frac{k_1}{1 + k_1 k_2}\,x_1\,.

This result is valid at all times, and we can obtain
:math:`x_3(t_n)` in terms of
:math:`x_1(t_n)` simply by evaluating the above formula.
Let us see if we can arrive at the same result by writing
out the equations for the respective blocks, as we did earlier.
Here are the equations we get:

.. math::
   :label: neq_alg_loop_3

   x_2^{n+1} = x_1^{n+1} - x_4^{n+1} \,,

.. math::
   :label: neq_alg_loop_4

   x_4^{n+1} = k_2\,x_3^{n+1} \,,

.. math::
   :label: neq_alg_loop_5

   x_3^{n+1} = k_1\,x_2^{n+1} \,.

As before, we wish to evaluate these formulas *one at a time*,
but this leads to a conflict.
Eqs. :eq:`neq_alg_loop_3` to :eq:`neq_alg_loop_5`
are supposed to be valid *simultaneously*. For example,
the value of
:math:`x_3^{n+1}` in
Eqs. :eq:`neq_alg_loop_4` and :eq:`neq_alg_loop_5`
is supposed to be the same. However, since
Eq. :eq:`neq_alg_loop_5` is evaluated *after*
Eq. :eq:`neq_alg_loop_4`, the two values are clearly different.
This type of conflict occurs when there is an **algebraic loop**
in the system, i.e., there is a loop in which the variables are
related through purely *algebraic* equations, not
involving time derivatives.

For an explicit method, an algebraic loop presents a roadblock.
We have the following options for a system with one or more
algebraic loops.

- If possible, avoid algebraic loops. In the above example, this
  would mean using
  :math:`x_3 = \displaystyle\frac{k_1}{1 + k_1 k_2}\,x_1`
  directly.

- Insert a delay element to "break" an algebraic loop as shown in
  the following figure.

  .. image:: alg_loop_delay.png
     :width: 600
     :alt: alg loop

  How does this help? The system equations can now be written as

  .. math::
     :label: neq_alg_loop_7

     \begin{align}
      x_4^{n+1} &= x_{3a}^n \,,\\
      x_2^{n+1} &= x_1^{n+1} + x_4^{n+1} \,,\\
      x_3^{n+1} &= k_1\,x_2^{n+1} \,,\\
      x_{3a}^{n+1} &= k_2\,x_3^{n+1} \,.
     \end{align}

  Note that :math:`x_{3a}` in the first equation has a superscript :math:`n`
  because of the delay element.
  We can see that the variables have now got "decoupled," allowing a sequential
  evaluation of the formulas, not requiring any iterative procedure. If the
  delay is small compared to the time constants in the overall system, this
  approach -- however crude it may sound -- gives acceptable accuracy although
  it does change the original problem to some extent.

- Treat the integrator block(s) first (since their outputs depend
  only on the past values), compute their outputs, and with those
  held constant, solve the system of equations representing the
  rest of the system. Let us illustrate this process with the
  following example.

  .. image:: flowgraph2.png
     :width: 300
     :alt: flow graph example 2

  We first update the integrator output as

  .. math::

     x_2^{n+1} = x_2^n + h\,x_1^n\,,

  where :math:`h` is the time step. Now, we treat 
  :math:`x_2^{n+1}` as a constant (say, :math:`c`), and assemble the equations as
  follows.

  .. math::
     :label: neq_alg_loop_6

     \begin{align}
      x_2^{n+1} &= c,\\
      x_1^{n+1} &= A_1 \sin \omega \, t_{n+1},\\
      x_3^{n+1} &= x_2^{n+1} + x_5^{n+1},\\
      x_4^{n+1} &= k_1\,x_3^{n+1},\\
      x_5^{n+1} &= k_2\,x_4^{n+1}.
     \end{align}

  We have now obtained a system of equations which can be solved to get
  updated values of
  :math:`x_1`,
  :math:`x_3`,
  :math:`x_4`,
  :math:`x_5`. Note that, in this example, the system of equations happens
  to be linear, but in the general case, it could be nonlinear.
- Use an implicit method; in that case, algebraic loops do not need any special
  treatment.

.. _implicit:

================
Implicit methods
================

We have discussed so far a few explicit methods (in particular,
the Forward Euler (FE), Runge-Kutta order 4 (RK4), and
Runge-Kutta-Fehlberg (RKF45) methods) for solving the ODE
:math:`\displaystyle\frac{dx}{dt} = f(t,x)`, with
:math:`x(t_0) = x_0`.
We have also seen how the RKF45 method -- a combination
of an RK4 method and an RK5 method -- can be used to control
the time step in order to maintain a given accuracy (tolerance).
There are several other explicit methods available in the
literature (see [Lapidus, 1971], for example).
We can summarise the salient features of explicit methods as follows.

- Explicit methods are easy to implement since they only
  involve *evaluation* of quantities rather than
  *solution* of equations. The computational effort
  per time point is therefore relatively small.
- Explicit methods can be extended to a system of ODEs in
  a straightforward manner. If there are :math:`N` ODEs, the
  computational effort would increase by a factor of :math:`N`
  as compared to solving a single ODE
  (assuming all ODEs to be of similar complexity).
- Explicit methods are conditionally stable -- if the time
  step is not sufficiently small (of the same order as the
  smallest time constant), the numerical solution can
  "blow up" (i.e., become unbounded).
- In some problems, it may be difficult to know the various time
  constants involved. In such cases, the "auto" (automatic or adaptive)
  time step methods such as RKF45 are convenient. These methods can adjust
  the time step automatically to ensure a certain
  accuracy (in terms of the local truncation error estimate) and in
  the process keep the numerical solution from blowing up.
- Explicit methods are not suitable for stiff problems in which there
  are vastly different time constants involved. This is because the
  stability constraints imposed by an explicit method force small
  time steps which may not be required from the accuracy perspective
  at all.

Given the above limitations, it is clear that an alternative
must be found for explicit methods. Implicit methods provide that
alternative.

.. _betrz:

BE, TRZ, and BDF2 methods
=========================

We will now look at the most commonly used implicit methods, viz.,
the backward Euler and trapezoidal methods. Systematic approaches are
available to derive these methods [Patil 2009]; here, we will present
a simplistic intuitive picture. Consider

.. math::
   :label: eq_impl_1

   \displaystyle\frac{dx}{dt} = f(t,x),~~~x(t_0) = x_0.

Suppose the solution :math:`x(t)` is given by the curve shown in
the following figure.

.. _figbetrz:

.. figure:: impl1.png
   :width: 350
   :alt: BE method

   Simple derivation of FE and BE formulas.

Consider the slope of the line joining the points
:math:`(t_n,x(t_n))` and
:math:`(t_{n+1},x(t_{n+1}))`, i.e.,
:math:`m = \displaystyle\frac{x(t_{n+1})-x(t_n)}{h}`.
The forward Euler (FE) method results if we approximate the slope as

.. math::
   :label: eq_impl_2

   \displaystyle\frac{x(t_{n+1})-x(t_n)}{h} 
   \approx
   \displaystyle\frac{x_{n+1}-x_n}{h} 
   \approx
   \left. \displaystyle\frac{dx}{dt}\right|_{(t_n,x_n)}
   = f(t_n,x_n),

where :math:`x_n` and :math:`x_{n+1}` are the numerical solutions at
:math:`t_n` and :math:`t_{n+1}`, respectively.

The backward Euler (BE) method results if the slope :math:`m` is
equated to the slope at 
:math:`(t_{n+1},x_{n+1})`, i.e.,
:math:`\left. \displaystyle\frac{dx}{dt}\right|_{(t_{n+1},x_{n+1})}`
rather than
:math:`\left. \displaystyle\frac{dx}{dt}\right|_{(t_n,x_n)}`.
In that case, we get

.. math::
   :label: eq_impl_3

   \displaystyle\frac{x(t_{n+1})-x(t_n)}{h} 
   \approx
   \displaystyle\frac{x_{n+1}-x_n}{h} 
   \approx
   \left. \displaystyle\frac{dx}{dt}\right|_{(t_{n+1},x_{n+1})}
   = f(t_{n+1},x_{n+1}),

leading to

.. math::
   :label: eq_impl_4

   x_{n+1} = x_n + h
   f(t_{n+1},x_{n+1}).

In case of the trapezoidal (TRZ) method, the slope :math:`m`
is equated to the average of the two slopes (at
:math:`(t_{n},x_{n})` and
:math:`(t_{n+1},x_{n+1})`), i.e.,

.. math::
   :label: eq_impl_5

   \displaystyle\frac{x(t_{n+1})-x(t_n)}{h} 
   \approx
   \displaystyle\frac{x_{n+1}-x_n}{h} 
   \approx
   \displaystyle\frac{1}{2}
   \left(
   \left. \displaystyle\frac{dx}{dt}\right|_{(t_{n},x_{n})}+
   \left. \displaystyle\frac{dx}{dt}\right|_{(t_{n+1},x_{n+1})}
   \right)
   = 
   \displaystyle\frac{1}{2}
   \left(
   f(t_{n},x_{n})+
   f(t_{n+1},x_{n+1})
   \right),

leading to

.. math::
   :label: eq_impl_6

   x_{n+1} = x_n + \displaystyle\frac{h}{2}
   \left(
   f(t_{n},x_{n})+
   f(t_{n+1},x_{n+1})
   \right).

Note that we have shown the time steps to be uniform in
:numref:`figbetrz`, but the above equations can also be
used for non-uniform time steps by replacing :math:`h` with
:math:`h_n \equiv t_{n+1}-t_n`.

The BE and TRZ methods are *implicit* in nature since
:math:`x_{n+1}` is involved in the right-hand sides of
Eqs. :eq:`eq_impl_4` and :eq:`eq_impl_6`. In other words,
:math:`x_{n+1}` cannot be simply *evaluated* in terms of known
quantities (except in some special cases such as
:math:`(a)\,f(t_{n+1},x_{n+1})` is linear in :math:`x_{n+1}`,
:math:`(b)\,f(t_{n+1},x_{n+1})` does not involve :math:`x_{n+1}` at all,
e.g., :math:`f(t,x) = \cos \omega t`).

Instead, it needs to be obtained by *solving*
Eq. :eq:`eq_impl_4` or Eq. :eq:`eq_impl_6`.
As an example, consider

.. math::
   :label: eq_impl_7

   \displaystyle\frac{dx}{dt} \,\equiv\, f(t,x) \,=\, \cos x,~~
   x(0) = 0.

The FE and BE methods give the discretised equations,

.. math::
   :label: eq_impl_8

   \begin{align}
    {\textrm{FE :}}~
    &
    x_{n+1} = x_{n} + h\,\cos\,(x_n),
    \\
    {\textrm{BE :}}~
    &
    x_{n+1} = x_{n} + h\,\cos\,(x_{n+1}).
   \end{align}

There is a fundamental difference between the two equations
in terms of computational effort. If we use the FE method,
:math:`x_{n+1}` can be obtained by simply evaluating the right-hand
side (since :math:`x_n` is known). With the BE method, the task is far more complex because
the presence of
:math:`x_{n+1}` on the RHS has made the equation nonlinear, thus requiring
an iterative solution process.
If we use the NR method, for example,
then each iteration will involve evaluation of the function,
its derivative, and the correction. Clearly, the work involved per
time point is much more when the BE method is used.
The following C program shows the implementation of the FE and BE
methods for solving Eq. :eq:`eq_impl_7`.

.. code-block:: text
   :linenos:

   #include<stdio.h>
   #include<stdlib.h>
   #include<math.h>

   int main()
   {
      double t,x,f,t_start,t_end,h;
      double x_old;
      double f_nr,dfdx_nr,delx_nr;
      int i_nr,nmax_nr=10;
      FILE *fp;

      t_start = 0.0;
      t_end = 8.0;
      h = 0.05;

   // Forward Euler:
      fp=fopen("fe.dat","w");

      t=t_start;
      x = 0.0; // initial condition
      fprintf(fp,"%13.6e %13.6e\n",t,x);

      while (t <= t_end) {
        f = cos(x);
        x = x + h*f;
        t = t + h;
        fprintf(fp,"%13.6e %13.6e\n",t,x);
      }
      fclose(fp);

   // Backward Euler:
      fp=fopen("be.dat","w");

      t=t_start;
      x = 0.0; // initial condition
      x_old = x;
      fprintf(fp,"%13.6e %13.6e\n",t,x);

      while (t <= t_end) {
   //   Newton-Raphson loop
        for (i_nr=0; i_nr < (nmax_nr+1); i_nr++) {
          if (i_nr == nmax_nr) {
            printf("N-R did not converge.\n");
            exit(0);
          }
          f_nr = x - h*cos(x) - x_old;
          if (fabs(f_nr) < 1.0e-8) break;
          dfdx_nr = 1.0 + h*sin(x);
          delx_nr = -f_nr/dfdx_nr;
          x = x + delx_nr;
        }
        t = t + h;
        fprintf(fp,"%13.6e %13.6e\n",t,x);
        x_old = x;
      }
      fclose(fp);
   }

The results are shown in the following figure for
a step size of :math:`h = 0.05`.

.. image:: impl3a.png
  :width: 400
  :alt: FE-BE results

The BE and TRZ methods can be extended to solve a set of
ODEs (see Eqs. :eq:`neq_ode_9`, :eq:`neq_ode_10`):

.. math::
   :label: eq_impl_9

   \begin{align}
   {\textrm{BE:}}~
   &
   {\bf{x}}^{(n+1)} = {\bf{x}}^{(n)} + h\,{\bf{f}}(t_{n+1},{\bf{x}}^{(n+1)}),
   \\
   {\textrm{TRZ:}}~
   &
   {\bf{x}}^{(n+1)} = {\bf{x}}^{(n)} + \displaystyle\frac{h}{2}
   \left(
   {\bf{f}}(t_{n},{\bf{x}}^{(n)})+
   {\bf{f}}(t_{n+1},{\bf{x}}^{(n+1)})
   \right),
   \end{align}

where
:math:`{\bf{x}}^{(n)}` stands for the numerical solution at :math:`t_n`, and so on.
As an illustration, let us consider the set of two ODEs
seen earlier (Eq. :eq:`neq_ode_13`) and reproduced here:

.. math::
   :label: eq_impl_10

   \begin{align}
   \displaystyle\frac{dx_1}{dt}
   &= a_1\left(\sin \omega t -x_1\right)^3-a_2\left(x_1-x_2\right),
   \\
   \displaystyle\frac{dx_2}{dt}
   &= a_3\left(x_1-x_2\right),
   \end{align}

with the initial condition,
:math:`x_1(0) = 0`,
:math:`x_2(0) = 0`.
The BE equations are given by

.. math::
   :label: eq_impl_11

   \begin{align}
   x_1^{(n+1)}
   &=
   x_1^{(n)} +
   h\left[a_1\left(\sin \omega t_{n+1} - x_1^{(n+1)}\right)^3-a_2\left(x_1^{(n+1)}-x_2^{(n+1)}\right)\right],
   \\
   x_2^{(n+1)}
   &=
   x_2^{(n)} +
   h\left[a_3\left(x_1^{(n+1)}-x_2^{(n+1)}\right)\right].
   \end{align}

We now have a set of nonlinear equations which must be solved
at each time point. The NR method is commonly used for this
purpose, and it entails computation of the Jacobian matrix and
the function vector, and solution of the Jacobian equation
(of the form :math:`{\bf{A}}{\bf{x}} = {\bf{b}}`) in each
iteration of the NR loop. As :math:`N` (the number of ODEs) increases, the computational
cost of solving the Jacobian equations increases superlinearly
(:math:`O(N^3)` for a dense matrix). On the other hand, with an explicit method,
we do not require to solve a system of equations, and the computational
effort can be expected to grow linearly with :math:`N` (see the FE equations given by
Eq. :eq:`neq_ode_14`, for example). With respect to computational effort,
explicit methods are therefore clearly superior to implicit methods
*if* the number of time points to be simulated is the same in
both cases. That is a big if, as we will soon discover.

Another implicit method commonly used in circuit simulation is the
Backward Differentiation Formula of order 2 (BDF2), also known as
Gear's method of order 2. It is given by

.. math::
   :label: eq_bdf2

   \displaystyle\frac{3}{2}\,x_{n+1}
   -2\,x_n
   +\displaystyle\frac{1}{2}\,x_{n-1}
   =h\,f(t_{n+1},x_{n+1}).

Note that the BDF2 method involves :math:`x_{n-1}` (in addition to
:math:`x_{n+1}` and :math:`x_n`), i.e., the numerical solution at
:math:`t_{n-1}`. In deriving Eq. :eq:`eq_bdf2`, the time step is
assumed to be uniform, i.e.,
:math:`t_{n+1}-t_n = t_n -t_{n-1} = h`; however,
it is possible to extend the BDF2 formula to the case of non-uniform
time steps [McCalla, 1987]. With the BDF2 method (methods involving multiple
steps in general), starting is an issue since
at the very beginning (:math:`t = t_0`), the solution is available
only at one time point, :math:`x(t_0)`. In practice, a single-step method such as the BE method
is used to first go from :math:`t_0` to :math:`t_1`.
The BDF2 method can then be used to compute
:math:`x_2` from :math:`x_0` and :math:`x_1`,
:math:`x_3` from :math:`x_1` and :math:`x_2`, and so on.

.. _implicit_stability:

Stability
=========

As we have seen in the :ref:`explicit_stability` section, the explicit methods
we have discussed, viz., FE and RK4, are conditionally stable. This
puts an upper limit :math:`h_{\mathrm{max}}` on the step size while
solving the test equation,

.. math::
   :label: eq_impl_12

   \displaystyle\frac{dx}{dt} = -\,\displaystyle\frac{t}{\tau},~~x(0)=1.

For the FE method,
:math:`h_{\mathrm{max}}` is :math:`2\,\tau`, and for the RK4 method, it is :math:`2.8\,\tau`.
Other explicit methods are also conditionally stable.

The FE method is a member of the
Adams-Bashforth (AB) family of explicit linear multi-step methods.
The regions of stability of the
AB methods with respect to Eq. :eq:`eq_impl_12` are shown in
:numref:`stability_ab_am` (a).
For the AB1 (FE) method, we require
:math:`h/\tau < 2`, as we have already seen. As the order increases,
the AB methods become more accurate, but the region of stability shrinks.

.. _stability_ab_am:

.. figure:: impl2.png
   :width: 650
   :alt: FE-BE results

   Regions of stability for A-B and A-M methods.

The BE and TRZ methods
belong to the Adams-Moulton (AM) family of implicit linear
multi-step methods. The BE method
is of order 1 while the TRZ method is of order 2.
The stability properties of the AM methods
with respect to Eq. :eq:`eq_impl_12` are shown in
:numref:`stability_ab_am` (b).
The BE and TRZ methods are
*unconditionally* stable while higher-order AM methods
have finite regions of stability which shrink as the order
increases. (Note: The BDF2 formula given by Eq. :eq:`eq_bdf2`
is also unconditionally stable, see [Patil, 2009]).

The unconditional stability of the BE and TRZ methods
allows us to break free from the stability
constraints which arise when an explicit method is used for a stiff circuit
(see :ref:`explicit_stability`). This is a *huge* benefit; it means
that, if we use the BE or TRZ method, the only restriction on the time
step comes from accuracy of the numerical solution and not from stability.
For example, let us re-visit the stiff circuit shown in the following figure with
:math:`V_m = 1\,{\textrm{V}}`,
:math:`f = 1\,{\textrm{kHz}}`. The time step is :math:`h = 0.02\,{\textrm{msec}}`.
We recall that, for this circuit, the RKF45 method was forced to
use very small time steps -- of the order of nanoseconds --
because of stability considerations. The BE method is
not constrained by stability issues, and therefore a much larger time
step (:math:`h = 0.02\,{\textrm{msec}}`) can be used
to obtain the numerical solution, as shown in the figure.

.. figure:: ode18a.png
   :width: 650
   :alt: stiff circuit and BE results

   Stiff circuit example

As another example, consider the half-wave rectifier circuit
shown below.

.. image:: impl4_cct.png
   :width: 200
   :alt: half-wave rectifier

The diode is represented with the
:math:`R_{\mathrm{on}}`/:math:`R_{\mathrm{off}}` model, with a turn-on voltage
equal to :math:`0\,{\textrm{V}}` and :math:`R_{\mathrm{off}} = 1\,{\textrm{M}}\Omega`.
The other parameters are
:math:`V_m = 1\,{\textrm{V}}`,
:math:`f = 50\,{\textrm{Hz}}`, :math:`R = 500\,\Omega`,
:math:`C = 600\,\mu {\textrm{F}}`.

The circuit behaviour is described by the ODE,

.. math::
   :label: eq_impl_13

   \displaystyle\frac{dV_o}{dt} =
   \displaystyle\frac{1}{C}\left[
    \displaystyle\frac{V_s-V_o}{R_D}
   -\displaystyle\frac{V_o}{R}
   \right],

where :math:`R_D` is the diode resistance
(:math:`R_{\mathrm{on}}` if :math:`V_s > V_o`;
:math:`R_{\mathrm{off}}` otherwise).
The numerical solution of the above ODE obtained with the RKF45 method
is shown below.

.. image:: impl4_plots.png
   :width: 550
   :alt: RKF45 results for half-wave rectifier

The charging and discharging intervals can be clearly identified --
charging takes place when the diode current is non-zero; otherwise,
the capacitor discharges through the load resistor.
When the diode conducts, the circuit time constant is
:math:`\tau _1= (R_{\mathrm{on}}\parallel R)\,C`
(approximately :math:`R_{\mathrm{on}}C`); otherwise it is :math:`\tau _2 =R\,C`.
The RKF45 method -- like other conditionally stable methods -- requires
that the time step be of the order of the circuit time constant.
Since :math:`\tau _ 1 \ll \tau _2`,
the time step used by the RKF45 algorithm is much smaller in the
charging phase compared to the discharging phase, as
seen in the figure. If we use a smaller value
of :math:`R_{\mathrm{on}}`, :math:`\tau _1` becomes smaller, and smaller time
steps get forced.

On the other hand, the BE or TRZ method would not be constrained by
stability considerations at all, and for these methods, a much larger
time step can be selected as long as it gives sufficient accuracy.
For the half-wave rectifier example, :math:`20\,\mu {\textrm{sec}}` is suitable,
*irrespective* of the value of :math:`R_{\mathrm{on}}`.

.. _sec_practical_issues:

Some practical issues
=====================

We have already covered the most important aspects of
numerical methods for solving ODEs: (a) comparison of
explicit and implicit methods with respect to computation
time, (b) accuracy (order) of a method, (c) constraints
imposed on the time step by stability considerations of a method.
In addition, we need to understand some specific situations which arise
in circuit simulation.

.. _sec_osc:

Oscillatory circuits
--------------------

Consider an :math:`LC` circuit without any resistance (see figure)
and with the initial conditions,
:math:`v_C(0) = 1\,{\textrm{V}}`,
:math:`i_L(0) = 0\,{\textrm{A}}`.

.. image:: impl8a.png
   :width: 650
   :alt: LC circuit

The circuit equations can be described by the following set of ODEs.

.. math::
   :label: eq_osc_1

   \begin{align}
   \displaystyle\frac{dv_C}{dt} &=
   -\displaystyle\frac{1}{C}\,i_L
   \,, \\
   \displaystyle\frac{di_L}{dt} &=
   \displaystyle\frac{1}{L}\,v_C
   \,,
   \end{align}

with the analytic solution given by

.. math::
   :label: eq_osc_2

   \begin{align}
   v_C &= \cos (\omega t) \,, \\
   i_L &= \sin (\omega t) \,,
   \end{align}

with :math:`\omega = 1/\sqrt{LC}`. We will consider :math:`L = 1\,{\textrm{H}}`,
:math:`C = 1\,{\textrm{F}}` which gives the frequency of oscillation
:math:`f_0 = 1/2\pi\,{\textrm{Hz}}`, i.e., a period :math:`T = 2\pi\,{\textrm{sec}}`.
The numerical results for :math:`v_C(t)` obtained with the
BE, TRZ, and BDF2 methods along with the analytic (exact) solution
are shown in the figure.
The TRZ method maintains the amplitude of :math:`v_C(t)` constant whereas
the BE and BDF2 methods lead to an artificial reduction (damping) of the
amplitude with time. Clearly, the BE and BDF2 methods are not suitable
for purely oscillatory circuits or for circuits with small amount of
natural damping. In such cases, the TRZ method should be used.

Although the TRZ method does not result in an amplitude error, it does
give a phase error, i.e., the phase of :math:`v_C(t)` differs from the expected
phase as time increases. To observe the phase error, expand the plot (zoom in),
and look at the exact and TRZ results; you will see the phase error growing
with time. The phase error can be reduced by selecting a smaller time step.

.. _sec_ringing:

Ringing
-------

Consider the RC circuit shown in the figure. The component values are
:math:`R_1 = R_2 = R_3 = 1\,\Omega`,
:math:`C_1 = C_2 = C_3 = 1\,{\textrm{F}}`,
:math:`V_C(0) = 0\,{\textrm{V}}` for all capacitors. A step
input going from :math:`0\,{\textrm{V}}` to :math:`1\,{\textrm{V}}` at :math:`t = 0` is applied.

.. image:: impl10a.png
   :width: 600
   :alt: ringing

The time constants for this circuit are 0.31, 0.64, and 5.05 seconds,
the smallest being :math:`\tau _{\mathrm{min}} = 0.31\,{\textrm{sec}}`.
BE and TRZ results with a relatively large time step are shown in the
figure. Sine the time step (:math:`5\,{\textrm{sec}}`) is much larger than
:math:`\tau _{\mathrm{min}}`, neither of the two methods can
track closely the expected solution.
However, there is a substantial difference
in the nature of the deviation of the numerical solution from the
exact solution -- With the TRZ method, the numerical solution
*overshoots* the exact solution, hovers around it in an
oscillatory manner, and finally returns to the expected value when
the transients have vanished. This phenomenon, which is specifically
associated with the TRZ method, is called "ringing." The BE result, in contrast, 
approaches the exact solution without overshooting.

Is ringing relevant in practice? Are we ever going to use a time step
which is much larger than :math:`\tau _{\mathrm{min}}`?
Yes, the situation does arise in practice, and we should therefore be
watchful. For example, in a typical power electronic circuit, there is frequent
switching activity. Whenever a switch closes, we can expect transients.
Since the switch resistance is small,
:math:`\tau _{\mathrm{min}}` is also small, typically much smaller than the time
scale on which we want to resolve the transient. In other words, in such cases,
the time step would be often much larger than
:math:`\tau _{\mathrm{min}}`, and we can expect ringing to occur if
the TRZ method is used. If there is a good reason for using the TRZ method
for the specific simulation of interest,
the time step should be suitably reduced in order to avoid ringing.

.. _sec_trbdf2:

TR-BDF2 method
==============

The TR-BDF2 method is a combination of TRZ and BDF2 methods.
In this method [Bank, 1992], the time interval :math:`h` from :math:`t_n`
to :math:`t_{n+1}` is divided into two intervals, as shown in the
figure.

.. image:: impl6.png
   :width: 350
   :alt: TR-BDF2 method

The TRZ method is used to go from :math:`x_n` to :math:`x_{n+\gamma}`
(i.e., the numerical solution at :math:`t_{n+\gamma} \equiv t_n + \gamma h`).
In the second step, the BDF2 method is used (using the three points,
:math:`t_n`, :math:`t_{n+\gamma}`, and :math:`t_{n+1}`),
to compute :math:`x_{n+1}`. The constant
:math:`\gamma` is selected such that the TRZ and BDF2
Jacobian matrices are identical (which means that the same
:math:`LU` factorisation can be used in the TRZ and BDF2 steps).
This leads to the following condition:

.. math::
   :label: eq_trbdf2_1

   \displaystyle\frac{2}{\gamma}
   = \displaystyle\frac{2-\gamma}{1-\gamma}\,,

or :math:`\gamma = 2-\sqrt{2} \approx 0.59`.

The TR-BDF2 method has the following advantages.

- It does not exhibit ringing.
- Although BDF2 is a two-step method,
  TR-BDF2 is a single-step method since the solution at
  :math:`t_{n+\gamma}` is an intermediate result in going from :math:`t_n`
  to :math:`t_{n+1}`. Therefore, no special procedure for starting
  is required.
- The TR-BDF2 method can be used for auto (adaptive) time stepping
  by estimating the LTE and comparing it with a user-specified
  tolerance.

.. _sec_mna_trns:

Systematic assembly of circuit equations
========================================

A circuit simulator like SPICE must be able to handle any general
circuit topology, and it should get the solution in an efficient
manner. In this section, we will see how a general circuit involving
time derivatives (e.g., in the form of capacitors and inductors)
is treated in a circuit simulator.

Consider the circuit shown below.

.. image:: impl11.png
   :width: 320
   :alt: circuit example

The following equations
describe the circuit behaviour, taking into account the relationship to
be satisfied for each element in the circuit.

.. math::
   :label: eq_dae_3

   \begin{align}
   C\,\displaystyle\frac{dv_C}{dt}
   & = &
   G_1(V_s-V_2) - i_L\,,
   L\,\displaystyle\frac{di_L}{dt}
   & = &
   V_3\,,
   i_L  & = & G_2(V_2-V_3)\,,
   \end{align}

where
:math:`G_1 = 1/R_1`.
:math:`G_2 = 1/R_2`.
Note that the above system of equation is a "mixed" system --
The first two equations are ODEs whereas the last equation is
an algebraic equation.
Such a set of equations is called "Differential-algebraic equations" (DAEs).
In some cases, it may be possible to manipulate the equations to
eliminate the algebraic equations and reduce the original set to an equivalent
set of ODEs.
However, that is not possible in general, especially when nonlinear terms are involved.

How did we come up these equations?
We knew that the equations for the capacitor current and inductor voltage must
be included somewhere. We looked at the circuit and found that there is a KCL
which involves the capacitor current. Similarly, there is a node voltage which is
the same as the inductor voltage. We then invoked the circuit topology and wrote
the equations such that they describe the circuit completely.
In other words, we used our *intuition* about circuits.
Unfortunately, this is not a *systematic* approach and is therefore of no
particular use in writing a general-purpose circuit simulator.
Instead of an *ad hoc* approach, we can use a systematic approach we
have already seen before -- the MNA approach --
and write the circuit equations as

.. math::
   :label: eq_mnatrns_1

   \begin{align}
   i_s + G_1(V_1-V_2) &= 0 \,, \\
   G_1(V_2-V_1) + G_2(V_2-V_3) + i_C &= 0 \,, \\
   G_2(V_3-V_2) + i_L &= 0 \,, \\
   V_1 &= V_s(t)\,,
   \end{align}

where the first three are KCL equations, and the last equation is
the element equation for the voltage source.
Our interest is in obtaining the solution of the above equations for
:math:`t_{n+1}` from that available for :math:`t_n`. Let us write the above
equations specifically for :math:`t = t_{n+1}`:

.. math::
   :label: eq_mnatrns_1a

   \begin{align}
   i_s^{n+1} + G_1(V_1^{n+1}-V_2^{n+1}) &= 0 \,, \\
   G_1(V_2^{n+1}-V_1^{n+1}) + G_2(V_2^{n+1}-V_3^{n+1}) + i_C^{n+1} &= 0 \,, \\
   G_2(V_3^{n+1}-V_2^{n+1}) + i_L^{n+1} &= 0 \,, \\
   V_1^{n+1} &= V_s(t_{n+1})\,.
   \end{align}

Next, we select a method for discretisation of the time derivatives.
As we have seen earlier, stability constraints restrict the choice to
the BE, TRZ, and BDF2 methods, all of them being implicit in nature.
Note that implicit Runge-Kutta methods are also unconditionally stable,
but they are suitable only when the circuit equations can be written
as a set of ODEs.

Suppose we choose the BE method. The capacitor and inductor currents
(:math:`i_C^{n+1}`, :math:`i_L^{n+1}`) are then obtained as

.. math::
   :label: eq_mnatrns_1b

   \displaystyle\frac{dV_2}{dt}
   = \displaystyle\frac{1}{C}\,i_C
   ~\rightarrow ~
   \displaystyle\frac{V_2^{n+1}-V_2^n}{h}
   = \displaystyle\frac{i_C^{n+1}}{C}
   ~\rightarrow~
   i_C^{n+1} = \displaystyle\frac{C}{h}\,(V_2^{n+1}-V_2^n)\,,

.. math::
   :label: eq_mnatrns_2b

   \displaystyle\frac{di_L}{dt}
   = \displaystyle\frac{1}{L}\,V_3
   ~\rightarrow ~
   \displaystyle\frac{i_L^{n+1}-i_L^n}{h}
   = \displaystyle\frac{V_3^{n+1}}{L}
   ~\rightarrow~
   i_L^{n+1} = i_L^n + \displaystyle\frac{h}{L}\,(V_3^{n+1})\,,

where :math:`h = t_{n+1}-t_n` is the time step. Substituting
for :math:`i_C^{n+1}` and :math:`i_L^{n+1}` in
Eq. :eq:`eq_mnatrns_1a`, we get

.. math::
   :label: eq_mnatrns_1c

   \begin{align}
   G_1 V_1^{n+1}-G_1 V_2^{n+1} + i_s^{n+1}
   &= 0 \,, \\
   -G_1 V_1^{n+1}
   +\left(G_1+G_2+\displaystyle\frac{C}{h}\right)V_2^{n+1}
   - G_2 V_3^{n+1}
   &= \displaystyle\frac{C}{h}\,V_2^n \,, \\
   -G_2 V_2^{n+1}
   +\left(G_2+\displaystyle\frac{h}{L}\right)V_3^{n+1}
   &= -i_L^n \,, \\
   V_1^{n+1}
   &= V_s(t_{n+1})\,,
   \end{align}

a linear system of four equations in four variables
(:math:`V_1^{n+1}`, :math:`V_2^{n+1}`, :math:`V_3^{n+1}`, :math:`i_s^{n+1}`).
It is now a simple matter of solving this
:math:`{\bf{A}}{\bf{x}} = {\bf{b}}` type problem to obtain the
numerical solution at :math:`t_{n+1}`.
We should keep in mind that solving :math:`{\bf{A}}{\bf{x}} = {\bf{b}}`
is conceptually simple but can take a significant amount of
computation time when the system is large.

What if there are nonlinear elements in the circuit? Let us consider
the circuit shown in the following figure.

.. image:: impl12.png
   :width: 250
   :alt: nonlinear circuit example

The diode is described by

.. math::
   :label: eq_diode_iv_1

   I_D = I_s\left(e^{V_D/V_T}-1\right).

We can write the MNA equations for this circuit as

.. math::
   :label: eq_mnatrns_nonlin_1

   i_s + i_C = 0,

.. math::
   :label: eq_mnatrns_nonlin_2

   -i_C -i_D + GV_2 = 0 ~ \rightarrow ~
   -i_C -I_s\left(e^{-V_2/V_T}-1\right) + GV_2 = 0,

.. math::
   :label: eq_mnatrns_nonlin_3

   V_1 = V_s(t),

with :math:`G = 1/R`. At :math:`t = t_{n+1}`, the equations are

.. math::
   :label: eq_mnatrns_nonlin_1a

   \begin{align}
   i_s^{n+1} + i_C^{n+1} &= 0, \\
   - i_C^{n+1} -I_s\left(e^{-V_2^{n+1}/V_T}-1\right) + GV_2^{n+1} &= 0, \\
   V_1^{n+1} &= V_s(t_{n+1})\,.
   \end{align}

The next step is to obtain :math:`i_C^{n+1}` using the BE approximation
for the capacitor equation:

.. math::
   :label: eq_mnatrns_nonlin_1b

   \displaystyle\frac{dv_C}{dt}
   = \displaystyle\frac{1}{C}\,i_C
   ~\rightarrow ~
   \displaystyle\frac{v_C^{n+1}-v_C^n}{h}
   = \displaystyle\frac{i_C^{n+1}}{C}
   ~\rightarrow~
   i_C^{n+1} = \displaystyle\frac{C}{h}\,\left[(V_1^{n+1}-V_2^{n+1})-(V_1^n-V_2^n)\right],

Finally, substituting for :math:`i_C^{n+1}` in
Eq. :eq:`eq_mnatrns_nonlin_1a`, we get

.. math::
   :label: eq_mnatrns_nonlin_1c

   \begin{align}
   i_s^{n+1}
   + \displaystyle\frac{C}{h} V_1^{n+1}
   - \displaystyle\frac{C}{h} V_2^{n+1}
   - \displaystyle\frac{C}{h}(V_1^n-V_2^n)
   &= 0, \\
   - \displaystyle\frac{C}{h} V_1^{n+1}
   + \displaystyle\frac{C}{h} V_2^{n+1}
   -I_s\left(e^{-V_2^{n+1}/V_T}-1\right) + GV_2^{n+1}
   + \displaystyle\frac{C}{h}(V_1^n-V_2^n)
   &= 0, \\
   V_1^{n+1} - V_s(t_{n+1}) &= 0.
   \end{align}

We now have a nonlinear set of three equations in three
variables (:math:`V_1^{n+1}`, :math:`V_2^{n+1}`, :math:`i_s^{n+1}`). Generally,
circuit simulators would use the NR method to solve 
these equations because of the attractive convergence
properties of the NR method seen earlier. Note that the
NR loop needs to be executed in *each* time step, i.e.,
the Jacobian equation
:math:`{\bf{J}}\Delta {\bf{x}} = -{\bf{f}}` must be solved
several times in each time step until the NR process converges.
Obviously, this is an expensive computation but it simply cannot be helped.
Some tricks may be employed to make the NR
process more efficient, e.g., :math:`{\bf{J}}^{-1}`
from the previous NR iteration can be used if :math:`{\bf{J}}` has
not changed significantly.

The benefits of the above approach (we will refer to it as the
"DAE approach") outweigh the computational complexity:

- Since unconditionally stable methods (BE, TRZ, BDF2) are used,
  stiff circuits can be handled without very small time steps.
- Algebraic loops do not pose any special problem since the 
  set of differential-algebraic equations is solved
  directly, without any approximations.

.. _sec_mna_auto_nr:

Adaptive time steps using NR convergence
========================================

As we have seen in the :ref:`nr` section, the initial guess
plays an important role in deciding whether the NR process
will converge for a given nonlinear problem. This feature can
be used to control the time step in transient simulation. The
solution obtained at :math:`t_n` serves as the initial guess for
solving the circuit equations at :math:`t_{n+1}`. If :math:`h \equiv t_{n+1}-t_n`
is sufficiently small, we expect the initial guess to work well,
i.e., we expect the NR process to converge. If :math:`h` is large,
the NR process may not converge, or may take a larger number
of iterations to converge. By monitoring the NR convergence process,
it is possible control the time step.

The flow chart for auto (adaptive) time steps is shown below.

.. image:: nrauto2.png
   :width: 600
   :alt: nonlinear circuit flowchart

The basic idea is to allow only a certain maximum number
:math:`N_{\mathrm{NR}}^{\mathrm{max}}`
of NR iterations at each time point. If the NR process does not
converge, it means that the time step being taken is too large,
and it needs to be reduced (by a factor :math:`k_{\mathrm{down}}`).
However, if the NR process consistently converges in less than
:math:`N_{\mathrm{NR}}^{\mathrm{max}}` iterations, it means that a small
time step is not necessary any longer, and we then increase
it (by a factor :math:`k_{\mathrm{up}}`).
In this manner, the step size is made small
when required but is allowed to become larger when convergence
is easier. In practice, this means that small time steps are
forced when the solution is varying rapidly, and large time
steps are used when the solution is varying gradually.

Application of the above procedure to an oscillator circuit
is shown in the following figure.
The parameter values are
:math:`R_1 = R_2 = 1\,{\textrm{k}}\Omega`,
:math:`R_0 = 100\,\Omega`,
:math:`C = 1\,\mu {\textrm{F}}`,
:math:`C_0 = 0.1\,\mu {\textrm{F}}`,
:math:`V_{IL} = 1 \,{\textrm{V}}`,
:math:`V_{IH} = 4 \,{\textrm{V}}`,
:math:`V_{OL} = 0 \,{\textrm{V}}`,
:math:`V_{OH} = 5 \,{\textrm{V}}`,
:math:`h_{\mathrm{min}} = 10^{-9} \,{\textrm{sec}}`,
:math:`h_{\mathrm{max}} = 10^{-4} \,{\textrm{sec}}`,
:math:`k_{\mathrm{up}} = 1.1`,
:math:`k_{\mathrm{down}} = 0.8`,
:math:`N_{\mathrm{NR}}^{\mathrm{max}} = 10`.

.. image:: nrauto1.png
   :width: 500
   :alt: nonlinear circuit flowchart

The output voltage :math:`V_4` controls the switch -- when :math:`V_4` is
high, the switch turns on; otherwise, it is off. Note that,
when :math:`V_4` changes from low to high (or high to low), small
time steps get forced. As :math:`V_4` settles down, the time steps
become progressively larger, capped finally by :math:`h_{\mathrm{max}}`.

==========
References
==========

- [Lapidus, 1971] L. Lapidus and J.H. Seinfeld,
  *Numerical Solution of Ordinary Differential Equations*.
  New York: Academic Press, 1971.
- [McCalla, 1987] W.J. McCalla,
  *Fundamentals of Computer-Aided Circuit Simulation*.
  Boston: Kluwer Academic, 1987.
- [Bank, 1992] R.E. Bank, W.M. Coughran, W. Fichtner, E.H. Grosse, D.J. Rose, and R.K. Smith,
  "Transient simulation of silicon devices and circuits",
  *IEEE Trans. Electron Devices*, vol. 32, pp. 1992-2006, 1992.
- [Shampine, 1994] L.F. Shampine, *Numerical Solution of Ordinary Differential Equations*.
  New York: Chapman and Hall, 1994.
- [Burden, 2001] R.L. Burden and J.D. Faires, *Numerical Analysis*. Singapore: Thomson, 2001.
- [Patil, 2009] M.B. Patil, V. Ramanarayanan, and V.T. Ranganathan,
  *Simulation of power electronic circuits*.
  New Delhi: Narosa, 2009.

