Enabling ``Valgrind`` in CCI
============================

What is `Valgrind <https://valgrind.org/>`__
--------------------------------------------

``Valgrind`` is a powerful tool for detecting memory leaks, memory errors, and
other issues in C and C++ programs. It works by dynamically analyzing the
execution of your program and providing detailed reports about memory-related
problems.

Benefits of Enabling ``Valgrind`` in CCI
----------------------------------------

Enabling ``Valgrind`` in CCI offers several advantages:

1. **Early Detection of Memory Issues**:

   ``Valgrind`` can catch memory leaks, invalid memory accesses, and other
   memory-related problems before they make their way into production, improving
   the overall stability of your software.

2. **Enhanced Code Quality**:

   Running Valgrind regularly helps maintain code quality and enforces good
   memory management practices.

3. **Cost-Efficient Bug Detection**:

   Identifying and fixing memory-related bugs early in the development process
   saves time and resources that would otherwise be spent on debugging and
   maintenance.

How to use ``valgrind`` in ``conde``
------------------------------------

`conde documentation <https://waders-infra.gitlabe2-pages.ext.net.nokia.com/conde/ut.html?highlight=valgrind#running-tests-under-valgrind>`__

How to Enable Valgrind CCI
--------------------------

To enable Valgrind in CCI, follow these steps:

Scenario 1: There is only one unit test binary and name is ``testrunner``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  Modify the ``component-ci-configs``, add ``runValgrind=true``

   `Example <https://gitlabe2.ext.net.nokia.com/rcp/tools/component-ci-configs/6tunnelmanager/-/merge_requests/2/diffs>`__

   CCI will execute ``Run Valgrind`` `stage
   <http://component-ci-jenkins.eecloud.dynamic.nsn-net.net:8080/job/component-ci/job/auditlog/>`__.

   In this stage will run command ``conde -n <namespace> valgrind``.

   `conde valgrind <https://gitlabe2.ext.net.nokia.com/waders-infra/conde/-/blob/master/conde#L1270>`__
   will execute the ``testrunner`` with ``valgrind``.

**Result in CCI**

``Run Valgrind`` stage is enabled.

-  `Example: auditlog <http://component-ci-jenkins.eecloud.dynamic.nsn-net.net:8080/job/component-ci/job/auditlog/>`__

Scenario 2: The UT has multi binaries
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

For example: the following subsystem has 2 UT entries in the makefile

.. code:: makefile

   check_PROGRAMS = \
       testrunner \
       testtplpluginfhssh

``conde valgrind`` only run ``testrunner``.

**Solutions**:

**Option 1**: Combine the 2 binaries into 1 binary and the binary name is
``testrunner``

`Example <https://gerrite1.ext.net.nokia.com/c/scm_rcp/6tunnelmanager/+/1565035/3/Makefile.am>`__

With this solution will faster build, faster test execution and simpler
makefile.

**Option 2**: Developer define custom script for the ``Run Valgrind`` stage

.. code:: bash

   runValgrind=true
   customScriptDoValgrind='<user defined script>'

Please check `custom-stage-actions <https://rcp.gitlabe2-pages.ext.net.nokia.com/tools/component-ci/users-guide.html#custom-stage-actions>`__
for detail.

**Use Custom Script Example**

-  `customScriptDoValgrind <https://gitlabe2.ext.net.nokia.com/rcp/tools/component-ci-configs/certstorage/-/blob/master/params.properties#L99>`__
-  `customScript-example <https://gitlabe2.ext.net.nokia.com/rcp-security/certstorage/-/blob/master/ci/build_and_run_ut.sh>`__

**Note**:

The ``customScript-example`` is not yet finalized and will change in the near
future.

With this solution, team has full control of their own pipelines and the team
doesn't need anything new from anyone else.

Developer can choose above 2 options base on own requirement, combine multi
binaries into 1 binary is the recommended solution.

Scenario 3: Use ``make check-valgrind`` to run valgrind by ``autoconfig-archive``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- Change code to support ``make check-valgrind``

Add ``BuildRequires: autoconf-archive`` in spec.

Modify configure.ac:

.. code:: bash

   AX_VALGRIND_DFLT(memcheck, on)
   AX_VALGRIND_DFLT(helgrind, off)
   AX_VALGRIND_DFLT(drd, off)
   AX_VALGRIND_CHECK()

Modify in each Makefile.am with tests:

.. code:: bash

   @VALGRIND_CHECK_RULES@
   VALGRIND_SUPPRESSIONS_FILES = suppressions.conf
   EXTRA_DIST = suppressions.conf

`Example bip <https://gerrite1.ext.net.nokia.com/c/scm_rcp/bip/+/1625360/>`__

`suppressions.conf` is a suppressions configuration file. The developer can use 
`VALGRIND_SUPPRESSIONS_FILES` to configure suppress file in Makefile.am.

This results in a "check-valgrind" rule being added. Running ``make check-valgrind`` in that
directory will recursively run the module’s test suite for valgrind memcheck tool.
The results will be output to test-suite-memcheck.log. The target will succeed if
there are zero errors and fail otherwise.

- ``conde valgrind`` will execute the ``make check-valgrind``.

Integrates ``valgrind`` to unit tests
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

With this solution whenever developer, ``conde`` or component CI runs
``make check``, unit tests are automatically run with ``valgrind``.

The benefit is:

-  Quick feedback cycle
-  Errors found by ``valgrind`` will never even reach component CI, because
   ``valgrind`` is constantly used by developer while developing the code.

The downside is:

To explicitly disable ``valgrind`` when it cannot be used (e.g. with sanitizers)
or when developer wants to do rapid TDD and get unit test results very quickly.

`Example <https://gitlabe2.ext.net.nokia.com/rcp-security/auditlog/-/tree/master/tst>`__

Suppressing False Positives
---------------------------

``Valgrind`` might report false positives, where it identifies issues that are
not actual bugs. These can be suppressed using a suppressions file.

Generating a Suppressions Config File
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Follow these steps to generate a suppressions configuration file:

1. **Identify False Positives**

   Run ``Valgrind`` on your code and carefully review the reported issues.
   Determine which issues are false positives and shouldn't trigger alerts.

2. **Create the File**

   Create a text file `tst/valgrind-suppressions.conf <https://gitlabe2.ext.net.nokia.com/rcp-security/auditlog/-/blob/master/tst/valgrind-suppressions.conf>`__,
   to contain your suppression rules.

   **Note**:

   ``valgrind`` use the suppress file by ``--suppressions=<filename>``
   command-line option, ``conde`` use ``tst/valgrind-suppressions.conf`` as the
   the suppression config file name, developer should align with ``conde
   valgrind``

3. **Write** `Suppression Rules <https://valgrind.org/docs/manual/manual-core.html#manual-core.suppress>`__

Tips for Using Valgrind
-----------------------

-  Treat Valgrind issues seriously. Investigate and fix any real memory problems
   that it detects.
-  Use ``--gen-suppressions=all`` help to generate suppression config.
-  Use `Parse_valgrind_suppressions.sh <https://wiki.wxwidgets.org/Parse_valgrind_suppressions.sh>`__
   helps to parse the suppression
-  Give proper name to suppress rule
-  Using ``NiceMock<>`` when you are not interested about extra calls to mock
   functions, see `NiceStrictNaggy <http://google.github.io/googletest/gmock_cook_book.html#NiceStrictNaggy>`__.
-  Avoid gmock's default print by teaching ``gmock`` how to print your values,
   see: `teaching-gmock-how-to-print-your-values <http://google.github.io/googletest/gmock_cook_book.html#teaching-gmock-how-to-print-your-values>`__
-  Don't copy the suppress config file from other subsystem, write the suppress
   config file by yourself.

References
----------

-  `Valgrind manual <https://valgrind.org/docs/manual/manual-core.html>`__
-  `Valgrind_Suppression_File_How_to <https://wiki.wxwidgets.org/Valgrind_Suppression_File_Howto>`__
-  `Code-review <https://gerrite1.ext.net.nokia.com/c/scm_rcp/6tunnelmanager/+/1565036>`__
-  `customScriptDoValgrind <https://gitlabe2.ext.net.nokia.com/rcp/tools/component-ci-configs/certstorage/-/blob/master/params.properties#L99>`__
-  `customScript-example <https://gitlabe2.ext.net.nokia.com/rcp-security/certstorage/-/blob/master/ci/build_and_run_ut.sh>`__
-  `autoconfig-archive <https://www.gnu.org/software/autoconf-archive/ax_valgrind_check.html>`__
