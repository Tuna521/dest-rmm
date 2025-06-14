.. SPDX-License-Identifier: BSD-3-Clause
.. SPDX-FileCopyrightText: Copyright TF-RMM Contributors.

****
CBMC
****

CBMC is a Bounded Model Checker for C and C++ programs. For details see
`CBMC Home`_

CBMC in RMM
===========

CBMC needs to be run on a C program that has a single entry point. To test all
the RMM ABI commands, for each command a testbench file is created. These files
are generated by a script offline from the RMM MRS (Machine Readable
Specification), and committed to the RMM repository under the folder
``tools/cbmc/testbenches``

.. note::

    Currently only a subset of the ABI calls have a testbench implemented. Also
    there are errors reported by CBMC for some of the testbenches. Further
    testbenches and fixes are expected to be added later.

These files contain asserts that correspond to
the Faliure and Success conditions defined in the RMM specification. To read on
further how such a file should be defined please refer to
`Writing a good proof`_

The recommended way for installing CBMC is via the pre-built package found at
the `github release page`_. The same page contains the instructions for
installing the different release packages.

An example install command for Ubuntu linux is

.. code-block:: bash

    dpkg -i ubuntu-20.04-cbmc-5.95.1-Linux.deb

The invocation of CBMC tool is integrated in the RMM CMake system. CBMC analysis
can be run by passing specific targets (detailed later) to the build command. to
make the targets available the RMM build must be configured with
``-DRMM_CONFIG=host_defcfg -DHOST_VARIANT=host_cbmc`` options.

The results of a CBMC run are generated in the
``${RMM_BUILD_DIR}/tools/cbmc/cbmc_${MODE}_results`` directory. There are 3
files, ``${TESTBENCH_FILE_NAME}.${MODE}.[cmd|error|output]`` generated for each
RMM ABI under test, each one containing the CBMC command line, the CBMC
executable's output to the standard error, and the output to the standard out
respectively. There is also a single ``SUMMARY.${MODE}`` file is generated for
each build.

For an example build command please refer to :ref:`RMM Build Examples`

The CMake system provides different modes in which CBMC can be called, along
with their respective build targets.

CBMC Assert
===========

In this mode CBMC is configured, so that it tries to find inputs that makes an
assertion in the code to fail. If there is such an input, then CBMC provides a
trace that leads to that assertion failure.

To use this mode the target ``cbmc-assert`` must be passed to the build command.

CBMC Analysis
=============

In this mode CBMC is configured to generate assertions for certain properties in
the code. The properties are selected so that for example no buffer overflows,
or arithmetic overflow errors can happen in the code. For more details please
refer to `Automatically Generating Properties`_.
Then CBMC is run in a configuration similar to the Assert mode, except that this
time traces are not generated.

To use this mode the target ``cbmc-analysis`` must be passed to the build
command.

CBMC Coverage
=============

This mode checks whether all the conditions for an ABI function are covered.
The pre and post conditions for the command are expressed as boolean values in
the testbench, and a ``__CPROVER_cover()`` macro is added for each condition
that is expressed with the pre and post conditions. CBMC is configured to try
to generate an input for each ``__CPROVER_cover()`` call that makes the code
reach that call.

To use this mode the target ``cbmc-coverage`` must be passed to the build
command.

.. note::

    For all the modes the summary files are committed in the source tree as
    baseline in ``tools/cbmc/testbenches_results/BASELINE.${MODE}``.

-----

.. _CBMC Home: https://www.cprover.org/cbmc/
.. _Writing a good proof: https://model-checking.github.io/cbmc-training/management/Write-a-good-proof.html
.. _github release page: https://github.com/diffblue/cbmc/releases
.. _Automatically Generating Properties: https://www.cprover.org/cprover-manual/properties/

