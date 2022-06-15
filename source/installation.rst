
==================
GSEIM Installation
==================

Currently, GSEIM is available only for Linux. Here are
the installation steps.

- Make sure you have ``python3``, ``numpy``, ``psutil``, ``matplotlib``,
  ``g++`` installed.
- Download the ``gseim_grc`` folder from the
  `GSEIM distribution <https://github.com/gseim/gseim>`_
  in your home directory.
- ``cd ~/gseim_grc/gseim/cppsrc``
- ``make -f mkgseim``
- ``make -f mkfilter_so``
- ``mv libfilter.so ~/gseim_grc/gseim/exec/.``
- ``cd ~/gseim_grc``
- ``python3 ~/gseim_grc/grc/scripts/run_gseim``
- The GSEIM GUI should show up. Change the window to full size;
  quit the GUI and run the above ``python3`` command again.
- Follow instructions in the :ref:`Getting Started <getting_started>` section.
