**********************
UT Review Guidelines
**********************

.. contents:: :local:

1. How to check Unit Tests 
############################

In order to help teams with reviews in unit tests, there are some methods 
provided for checking unit tests validity or if it is really covered in the 
new code lines. These could be easy for reviewers to see whether the unit tests 
are really passed or not:


1.1. Using CQG to check unit test coverage
*********************************************

The purpose of checking unit tests coverage is not coverage itself. Instead,
by looking the coverage data to confirm whether the unit tests of new code are
covered, it can lighten reviewers' effort for reviewing unit tests.
For more info where to find it look at
`Code Quality Guardian <http://cqg.dynamic.nsn-net.net/ci3/dashboard/?project=rcp2.0_trunk>`_.

.. _Run unit tests with a random order:

1.2. Using test tools to check unit tests validity
**************************************************

It is mandatory to run unit tests normally. For C/C++ subsystems, we can use 
gtest to check the unit tests' results. And running the test cases with a random 
order is highly recommended. Some test cases may use the global variables, with a 
sequential execution, some issue might hide quietly. And the random order testing
way is pretty easy, you can refer to the following steps:
::

  git clone ssh://gerrite1.ext.net.nokia.com:8282/scm_rcp/rcplibdha-cpp
  cd rcplibdha-cpp  
  conde build  
  conde compile  
  conde check  
  conde shell  
  ./testrunner --gtest_shuffle  

If you want to execute the unit test cases repeatedly with a random order. You can
use the following commands:
::

./testrunner --gtest_shuffle --gtest_repeat=[COUNT]

By default, googletest uses a random seed calculated from the current time. Therefore you'll
get a different order every time. The console output includes the random seed value, so that you can
reproduce an order-related test failure later. To specify the random seed explicitly, you can use
:boldred:`\-\-gtest_random_seed`

If you feel inconvenient for inputting execution parameters every time, you can set the gtest
environment variables in your local environment. If defined, they have the same effect as
command line parameters, and you don't need to input the command line parameters anymore.
Here are some gtest environment variables you can refer to:

.. list-table::
   :header-rows: 1

   * - **environment variable**
     - **command line**
     - **variable value**
   * - GTEST_SHUFFLE
     - \-\-gtest_shuffle
     - GTEST_SHUFFLE=1 means enable shuffle, GTEST_SHUFFLE=0 means disable shuffle
   * - GTEST_REPEAT
     - \-\-gtest_repeat
     - GTEST_REPEAT=100 means repeatedly execute 100 times
   * - GTEST_RANDOM_SEED
     - \-\-gtest_random_seed
     - GTEST_RANDOM_SEED=0 means random seed is from the current time, you can set the seed as an integer in the range [0, 99999].


More info about gtest, you can use the command :boldred:`./testrunner \-\-help`

For Go subsystems, we use go test to run the test cases. And we can use 
:boldred:`go test -run=packageName -parallel=runningTimes` to run test cases in
a random order. :boldred:`packageName` is the name of go code package, and the 
:boldred:`runningTimes` is the number of test times for the test cases.


2. Guidelines for reviewers
###########################

1. :boldgreen:`No any test leak mock state (problem of singletons), it will make
   EXPECT_CALL() in gtest don't work. That means in gtest, you make sure that the
   mock object will be released at the end of test, or it may miss some bugs. For
   example, if one expectation is not called in your test execution, it should be
   a bug, and the result of testcase should be a failure. But if mock object is not
   released at the end of test, your testcase result might still be passed. The reason
   is that only when mock object is releasing, your friendly mock object will automatically
   verify that all expectations on it have been satisfied, and will generate Google Test
   failures if not. For more info, please refer to:`

   `Forcing a Verification`
   `<https://chromium.googlesource.com/external/github.com/google/googletest/+/refs/tags/release-1.8.0/googlemock/docs/CookBook.md>`_

   `Using Mocks in Tests`
   `<https://github.com/google/googletest/blob/master/docs/gmock_for_dummies.md>`_

2. :boldred:`Make sure the functions' return states is really checked.` :boldgreen:`As long
   as the function called in UT has a return value, developers should checking the
   functions' state, even if unit tests work well. Testcases are not just used to satisfy
   ut coverage`
3. :boldred:`Make sure that whether the unit test is missing for the modified function.`
   :boldgreen:`It is a good way to use CQG to check this point.`
4. :boldred:`Make sure  the unit test really cover the function.` :boldgreen:`For instance, the
   function may have three different states, but only one has been added unit test, so this
   case is not ut checking enough`
5. :boldred:`Don't use global mock objects".` :boldgreen:`For example, in case some socket functions
   like read or close mocked in testrunner, openssl will use such mocked functions by default.
   If the mock object is a global variable, main problem with global variables in C++ is that compilers
   don't guarantee initialization order. So the initialization order of openssl & gmock shared
   libraries is random, and you will never know that whether gmock shared library is ready for openssl
   to call. Therefore when you execute the testrunner which uses a global mock object, may get
   such error message:`

   ::

      Program received signal SIGSEGV, Segmentation fault.
      0x00007ffff7f942af in testing::internal::UntypedFunctionMockerBase::UntypedInvokeWith(void*) ()
      from /lib64/libgmock.so.1.10.0
      Missing separate debuginfos, use: dnf debuginfo-install SS_AsyncCommUtil-libs-1.4.1-1.wf33.x86_64 gmock-1.10.0-3.wf33.x86_64 gtest-1.10.0-3.wf33.x86_64 libcurl-minimal-7.71.1-8.wf33.x86_64 libevent-2.1.8-10.wf33.x86_64 libgcc-10.2.1-9.wf33.x86_64 libnghttp2-1.43.0-1.wf33.x86_64 libstdc++-10.2.1-9.wf33.x86_64 lz4-libs-1.9.1-3.wf33.x86_64 openssl-libs-1.1.1i-3.wf33.x86_64 rcphalog-libs-1.9.0-1.wf33.x86_64 rcplibdhadb-libs-1.51.0-1.wf33.x86_64 rcplibdvm-libs-1.30.0-1.wf33.x86_64 rcplibdvmdb-libs-1.32.0-1.wf33.x86_64 rcplibhadb-etcd-libs-1.15.0-1.wf33.x86_64 rcploglib-libs-1.7.2-1.wf33.x86_64 systemd-libs-246.10-1.wf33.x86_64 xz-libs-5.2.5-4.wf33.x86_64 zlib-1.2.11-23.wf33.x86_64
      (gdb) bt
      from /lib64/libgmock.so.1.10.0
       at /usr/include/gmock/gmock-spec-builders.h:1599
      0  0x00007ffff7f942af in testing::internal::UntypedFunctionMockerBase::UntypedInvokeWith(void*) ()
      from /lib64/libgmock.so.1.10.0
      1  0x00005555555a5685 in testing::internal::FunctionMocker<int (int)>::Invoke(int) (args#0=3, this=<optimized out>)
       at /usr/include/gmock/gmock-spec-builders.h:1599
      2  Mock_func_interface::close (gmock_a1=3, this=<optimized out>) at agent/tst/redundancy_socket_test/gmock_func.hpp:77
      3  close (fd=3) at agent/tst/redundancy_socket_test/stub.cpp:445
      4  0x00007ffff74670e5 in OPENSSL_init_library () from /lib64/libcrypto.so.1.1
      5  0x00007ffff7fe18de in call_init.part () from /lib64/ld-linux-x86-64.so.2
      6  0x00007ffff7fe19c8 in _dl_init () from /lib64/ld-linux-x86-64.so.2
      7  0x00007ffff7fd20ca in _dl_start_user () from /lib64/ld-linux-x86-64.so.2
      8  0x0000000000000001 in ?? ()
      9  0x00007fffffffe18e in ?? ()
      10 0x0000000000000000 in ?? ()
  
   :boldgreen:`In case you want to declare a global mock object in your testrunner. Then
   you need to make sure the testrunner will not lead to crash due to the random initialization order
   between different libraries. Here is an example:`
   `<https://gerrite1.ext.net.nokia.com/gitweb?p=scm_rcp/trs-pm-collector.git;a=commit;h=8970b92b16a16c5dd7851b71944921d4d0ddecbb>`_
   


3. Suggested learning material
##############################

Following trainings and instructions are useful:

1. How to get started in googletest https://github.com/google/googletest/blob/master/docs/primer.md
2. Gtest official guide about Googletest Mocking (gMock) Framework https://github.com/google/googletest/blob/master/googlemock/README.md
3. Advanced googletest Topics https://github.com/google/googletest/blob/master/docs/advanced.md
4. Go test official guide about how to use package testing https://golang.org/pkg/testing/
5. Go test official guild about how to use go test commands https://golang.org/pkg/cmd/go/internal/test/

