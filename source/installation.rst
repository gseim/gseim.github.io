
==================
GSEIM Installation
==================

Currently, GSEIM is available only for Linux. Here are
the installation steps.

- Make sure you have ``python3`` and ``g++`` installed.
- Download the ``gseim_gui`` folder from the
  `GSEIM distribution <https://github.com/gseim/gseim>`_
  in your home directory.
- ``cd ~/gseim_gui/gseim/cppsrc``
- ``make -f mkgseim``
- ``make -f mkfilter_so``
- ``mv libfilter.so ~/gseim_gui/gseim/exec/.``
- ``cd ~/gseim_gui``
- ``python3 ~/gseim_gui/grc/scripts/run_gseim``
- The GSEIM GUI should show up. Change the window to full size;
  quit the GUI and run ``run_gseim`` again.
- Follow instructions in the :ref:`Getting Started <getting_started>` section.
