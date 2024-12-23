********************
RCP1.0 UT guidelines
********************

.. contents:: :local:

The `RCP1.0` is legacy RCP release used by `ASRNC` product. There is very
little new feature development in `RCP1.0` but we need to maintain it until
`ASRNC` product will reach `EOL` probably somewhere in 2030.

The maintenance includes mainly updates and upgrades of used FOSS packages,
but also occasional new features. In order to be able to guarantee the quality
we also need to keep the UTs running.

The `RCP1.0` subsystems use following UT frameworks:

- `CppUnit`_
- `CppUTest`_
- `gtest`_
- `SS_UnitTest`
- `Cantata`_ (obsolete)

.. _CppUnit: https://freedesktop.org/wiki/Software/cppunit/
.. _CppUTest: http://cpputest.github.io/
.. _gtest: https://github.com/google/googletest
.. _Cantata: https://www.qa-systems.com/tools/cantata/

The `Cantata`_ is a commercial software. It is not available for RCP and if
used anywhere still needs to be replaced. This was used in FlexiPlatform at
some point where from RCP inherited many subsystems.

Running unit tests
##################

The unit tests can be run either in `RCP1.0` development image or using
`conde`. The unit tests in `RCP1.0` are run through `test` `make` target.
This target is also used by `FCI1`. For example:

::

        cd <subsystem>/build
        make test

If the dependencies are defined correctly they are built before running
actual build.

Also coverage data are generated and collected if subsystem is configured
properly. This is used by `FCI1` together with `CQG` to check that defined
coverage target was reached.


CppUnit
#######

The `CppUnit`_ is used widely in `RCP1.0` subsystems. Example use in `Makefile`:

::

        include $(VOBTAG)/flexiserver/build/cppunit.mk

        INCFLAGS += $(CPPUNIT_CFLAGS)

        APPLIBS  += $(CPPUNIT_LIBS)


CppUTest
########

Another popular framework in `RCP1.0`. Example use in `Makefile`:

::

        include $(VOBTAG)/flexiserver/build/cpputest.mk

        INCFLAGS += $(CPPUTEST_CFLAGS)

        APPLIBS  += $(CPPUTEST_LIBS)


gtest / gmock
#############

Number of subsystems uses in `RCP1.0` `gtest`_. Usually also `gmock` is used
at the same time. Example use in `Makefile`:

::

        include $(VOBTAG)/flexiserver/build/gtest.mk

        INCFLAGS += $(GTEST_CFLAGS) $(GMOCK_CFLAGS)

        APPLIBS  += $(GTEST_LIBS) $(GMOCK_LIBS)
        
If the `gtest_main` is used then link against `$(GTEST_MAIN_LIBS)`.


SS_UnitTest
###########

This subsystems contains wrappers for other frameworks. Some of the subsystems
uses the wrappers for example to get expected coverage data. The `SS_UnitTest`
also contains convenience utilities for port reservation used by tests requiring
`LDAP`. Example use in `Makefile`:

::

        include $(VOBTAG)/flexiserver/build/cppunit.mk
	include $(VOBTAG)/flexiserver/build/gtest.mk


        INCFLAGS += -I$(VOBTAG)/SS_UnitTest/include
        INCFLAGS += $(CPPUNIT_CFLAGS)

        APPLIBS  += -L$(VOBTAG)/SS_UnitTest/lib -lcppunit_runner
        APPLIBS  += $(CPPUNIT_LIBS)
        APPLIBS  += $(GTEST_LIBS) $(GMOCK_LIBS)

Even though only for example `CppUnit`_ is used you still must link also `gtest`_
libraries as the wrappers contain code for both frameworks in same module.
