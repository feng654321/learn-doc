************************
Unit tests best practice
************************

.. contents:: :local:

Writing readable unit tests
###########################

Code is the only document which include all accurate and actual information, so it's very important
to write readable code. Good readable code make developer or programmer easier to understand the
logic and maintain.

Readability is also very important for test code same as business code, it is the first thing you 
need consider when you write test code. Just by looking at the suite of unit tests, you should be 
able to infer the behavior of your code without even looking at the code itself. Additionally, when 
tests fail, you can see exactly which scenarios do not meet your expectations.


Giving good names for unit tests
--------------------------------

Good test name should have three parts:

- The name of the method being tested.
- The scenario under which it’s being tested.
- The expected behavior when the scenario is invoked.

One good way of naming unit test is using way of `Given-When-Then`_.

.. _Given-When-Then: https://proandroiddev.com/given-when-then-our-testing-approach-c9087b291c36

Given describes the preconditions and initial state before the start of a test and allows for any 
pre-test setup that may occur. When describes actions taken by a user during a test. Then describes
the outcome resulting from actions taken in the when clause. 

For bad example:

.. code-block:: c++

    TEST(Utils_test, getStringFromSet_test)
    {
        Utils Utils_ts;
        std::set<std::string> aSet;
        aSet.insert("one");
        aSet.insert("three");
        aSet.insert("two");
        std::string result = Utils_ts.getStringFromSet(aSet);
        STRCMP_EQUAL(result.c_str(),"one,three,two");
    }

    TEST(Upload_ConfigMan_Test, remove_upload_file_test)
    {
        ......
    }

    TEST(file_config_test, setContentByKeyword_test)
    {
        ......
    }

    func Test_ReadConfigurationWithValue(t *testing.T) {
        ......
    }

For good example:

.. code-block:: c++

    TEST_F(ipconfigureTest, FailedToGetIpv4AddressWhenHostnameEnvironmentVariableEmpty)
    {
        mockExpectCallNotConfigureIpRoleEnvironmentVariable();
        mockExpectCallMultiTimesNotConfigureHostnameEnvironmentVariable(2);

        expectFailedToGetIpAddress();
    }

    TEST(LoadWritePluginTest, IfSuitablePluginIsNotFoundThenLoadWritePluginThrows)
    {
        createPlugin = false;
        EXPECT_EQ(0, setenv(AUDITLOG_PLUGINS_ENV_VAR_NAME,
                            ".libs/libtestwriteplugin.so", // our test plugin
                            1));

        EXPECT_THROW(loadWritePlugin("/do/not/use/installed/plugins"), WritePluginImplementationNotFound);

        EXPECT_EQ(0, unsetenv(AUDITLOG_PLUGINS_ENV_VAR_NAME));
        createPlugin = true;
    }

    TEST(LinuxAuditLogParserTest, IfOpFieldIsMissingThenParseLinuxAuditLogReturnsNothing)
    {
        const std::string message("AUDIT1112 pid=7068 uid=0 auid=4294967295 ses=4294967295 msg='acct=\"UNKNOWN\" exe=\"/usr/bin/login\" hostname=hostname addr=? terminal=/dev/ttyS0 res=failed'");
        const ParsedLog parsedLog = {
            .timestamp = timestamp::Timestamp::now(),
            .level = LOG_NOTICE,
            .hostname = "hostname",
            .ident = "audit",
            .pid = 7068,
            .messageBegin = message.data(),
            .messageEnd = message.data() + message.size(),
        };
        EXPECT_FALSE(static_cast<bool>(parseLinuxAuditLog(parsedLog)));
    }

Giving good name for variables
------------------------------

How you name variables in unit tests is as important as or even more important than
variable-naming conventions in production code. By giving variables good
names, you can make sure that people reading your tests understand what you’re trying
to prove as quickly as possible.

- Use Intention-Revealing Names
- Avoid Disinformation
- Use Pronounceable Name
- Use Search-able Name
- Avoid Magic Strings and Numbers
- Avoid Abbreviation or spelling mistake

Bad example with magic number:

.. code-block:: c++

    TEST(Logger, BadlyNamedTest)
    {
        LogAnalyzer log;
        int result= log.GetLineCount("abc.txt");
        EXPECT_EQUAL(-100,result); //-100 is magic number
    }

Good option after re-factoring:

.. code-block:: c++

    TEST(Logger, GetLineCount_read_file_failure)
    {
        LogAnalyzer log;
        int result= log.GetLineCount("abc.txt");
        const int COULD_NOT_READ_FILE = -100;
        EXPECT_EQUAL(COULD_NOT_READ_FILE,result);
    }

Bad example with Abbreviation:

.. code-block:: c++

    bool excep = false;
    try {
        ......

    } catch (std::exception& ex) {
        excep = true;
    }
    CHECK_FALSE(excep);

Good option after re-factoring:

.. code-block:: c++

    bool isExceptionThrown = false;
    try {
        ......

    } catch (std::exception& ex) {
        isExceptionThrown = true;
    }
    CHECK_FALSE(excep);    

More bad examples:

.. code-block:: c++

    AddEvent addEvent; //It does not take any useful information, just repeat the class name.
    addEvent.m_dn = "fsipNetworkObjectName=FSIPIface@vivek,fsFragmentId=Network,fsipHostName=CLA-0,fsFragmentId=Nodes,fsFragmentId=HA,fsClusterId=ClusterRoot";

    AttributeValue attributeValue1; //non meaningful naming 
    attributeValue1.m_value = "RTE";
    Attribute attribute1;
    attribute1.m_description = "fsipEthernetIfaceMAC";
    attribute1.values().push_back(attributeValue1);

    addEvent.attributeList().push_back(attribute1);

    AttributeValue attributeValue2; //non meaningful naming
    attributeValue2.m_value = "up";
    Attribute attribute2;
    attribute2.m_description = "fsipIfaceAdminState";
    attribute2.values().push_back(attributeValue2);

    addEvent.attributeList().push_back(attribute2);


    //non meaningful name, and also these names are so strange
    MAP_METRIC_CAT wantResult;
    MAP_CAT_ACC mapCatAcc;
    MAP_ACC_AGG mapAccAgg;
    u64 aggValue = 25;
    mapAccAgg.insert(std::make_pair("/avg", aggValue));
    mapCatAcc.insert(std::make_pair("/as-0", mapAccAgg));
    wantResult.insert(std::make_pair("timeTriggered", mapCatAcc));

    //non meaningful name
    Event e1 = Event("ueload_overloaded_20percent", "ue", "", time(0)+ 1000, "");
    Event e2 = Event("cpuload_overloaded_20percent", "ue", "", time(0)+ 1000, "");

    //miss-leading naming
    std::vector<std::string> domainsNotDefault = {"default", "222"};
    constructFakeDomainList(domainsNotDefault);

.. code-block:: c++

One unit test just have one concern
-----------------------------------

Naming a test case may seem like a simple task, but if you’re testing more than one
thing, giving the test case a good name that indicates what’s being tested becomes almost
impossible. You end up with a very generic test case name that forces the reader to read the
test code. When you test just one concern, naming the test is easy.

One concern means that the checkpoint is only one error scenario or one output from function
or method.

Bad examples:

.. code-block:: c++

    TEST(pm_config_para_test, set_test)
    {
        PmConfigPara::getInstance()->setNotificationReportingFileThreshold(100);
        PmConfigPara::getInstance()->setNotificationReportingInterval(300);

        bool valid = PmConfigPara::getInstance()->checkValidNotificationReportingFileThreshold(100);
        CHECK(valid == true);
        valid = PmConfigPara::getInstance()->checkValidNotificationReportingInterval(300);
        CHECK(valid == true);

        valid = PmConfigPara::getInstance()->checkValidNotificationReportingInterval(301);
        CHECK(valid == false);
    }

    TEST(file_config_test, setContentByKeyword_test)
    {
        //error json
        //ConfigManagement::Status_t status = ConfigManagement::getInstance().setContentByKeyword("testkey", "{\"testkey\":\"test data\"}");
        ConfigManagement::Status_t status = ConfigManagement::getInstance()->setContentByKeyword("testkey", "invalid string");
        CHECK(status == ConfigManagement::ERROR);

        //empty content
        status = ConfigManagement::getInstance()->setContentByKeyword("testkey", "{}");
        CHECK(status == ConfigManagement::ERROR);

        //keyword error
        status = ConfigManagement::getInstance()->setContentByKeyword("testkey", "{\"testkey1\":\"test data\"}");
        CHECK(status == ConfigManagement::ERROR);

        status = ConfigManagement::getInstance()->setContentByKeyword("testkey", "{\"testkey\":\"test data\"}");
        CHECK(status == ConfigManagement::OK);

        //test save data to config file
        status = ConfigManagement::getInstance()->saveALLConfigs("./test_config.cfg");
        CHECK(status == ConfigManagement::OK);

        //test load data from config file
        status = ConfigManagement::getInstance()->loadAllConfigs("./test_config.cfg");
        CHECK(status == ConfigManagement::OK);
    }

You can't get any thing help from name of above unit tests, you need read test case implementation directly. Some time
it's not enough for reading test case implementation, you need to read source code further to get real intention of checkpoints.

Good examples:

.. code-block:: c++

    TEST_F(NotificationReceiverImplTest, ConfigChangedCbIsCalledWhenConfigIsUpdatedForTheFistTime)
    {
        const Config expectedConfig{true, "foo", false, ""};
        expectConfigChangedCb(expectedConfig);
        rcphttpcommon::HttpRequest request;
        request.body = configAndClientId2Json(expectedConfig, notificationReceiverImpl->getClientId());
        EXPECT_EQ(204, savedUrlActionCallback(std::error_code(), request, {}).statusCode);
    }

    TEST_F(NotificationReceiverImplTest, IfClientIdDoesNotMatchTheHttpRequestIsIgnored)
    {
        expectNoConfigChangedCb();
        rcphttpcommon::HttpRequest request;
        request.body = configAndClientId2Json(Config(), "wrongClientId");
        EXPECT_EQ(400, savedUrlActionCallback(std::error_code(), request, {}).statusCode);
    }

    TEST_F(ReaderImplTest, IfAsyncReaderReturnsErrorThenGetConfigThrows)
    {
        expectGetConfigAsync(Status::TIMEOUT, Config());
        EXPECT_THROW(readerImpl.getConfig(), Reader::Timeout);
    }

Avoid logic in test body
------------------------

A test that contains logic is not recommended, because it makes test
less readable, more fragile and therefore less maintainable. Anything
more complex causes the following problems:

- The test is harder to read and understand.
- The test is hard to re-create. (Imagine a multi-threaded test or
  a test with random numbers that suddenly fails.)
- Naming the test may be harder because it does multiple things.
- The test is more likely to have a bug or to test the wrong thing.

The chances of having bugs in your tests increase almost exponentially
as you include more and more logic in them. The worst thing you have done
is that you had a bug in your test since that is the hardest to be found.
It's hard to be found because you usually assume your test is OK and unless
you are doing true TDD(Test Driven Development), you are likely to find
that bug much later in time after you've searched for the possible problem
in the production code under test.

Also, having logic in a test can usually mean that you are testing more than
one thing – in that case, it is better to separate the single test into
multiple different tests, each testing a different aspect of the original test.
That would also make the test naming easier.

Possible types of logic inside a test may include:

- If-Else
- Switch-Case
- Loops
- Try-Catch
- `Repeat same production code logic in test`_

.. _Repeat same production code logic in test: https://livebook.manning.com/book/the-art-of-unit-testing-third-edition/chapter-7/v-8/64

Reference:

- `Book:the-art-of-unit-testing`_
- `Article:avoid-logic-inside-a-unit-test`_

.. _Book:the-art-of-unit-testing: https://livebook.manning.com/book/the-art-of-unit-testing-third-edition/chapter-7/v-8/57
.. _Article:avoid-logic-inside-a-unit-test: https://www.typemock.com/avoid-logic-inside-a-unit-test/

Avoid many mocks in one test cases
----------------------------------


Avoid big setup and teardown method
-----------------------------------

As everyone knows, `Setup method` is a special kind of method that is executed
by unit testing tools before each unit test in the suite. Such methods are commonly
used to set the stage before a unit test begins.
Analogously, `Teardown method` is a method that always run after unit test execution
and is usually used to perform cleanup after the test finishes.

As to Setup, there seems to be a number of reasons why you need to do things
before a test is actually run. For example, construction of object state to
prepare for the test (for instance setting up a Dependency Injection framework).
This is a valid reason for a setup, but could just as easily be done with a factory.

If your tested class have much more dependencies for other classes and libraries,
maybe you need initialize all mocks or stubs and also set some parameters in setup
method, clear the resources in teardown method, this will let these two methods
less readable.

When you need to write per-test set-up and tear-down logic, you have the choice
between using the test fixture constructor/destructor or SetUp()/TearDown().
The former is usually preferred.
Refer to `Should I use the constructor/destructor of the test fixture or SetUp()/TearDown()?`_
for more details.

The majority of valid uses for setup and teardown methods can be written as factory
methods (pack these mocks or stubs to a test class, the mocks, stubs and parameters
will be initialized in class constructor, and resource released in destructor) which
allows for DRY (`Don't Repeat Yourself`_) without getting into issues that seem to be
plagued with the setup/teardown paradigm.

.. _Should I use the constructor/destructor of the test fixture or SetUp()/TearDown()?: http://google.github.io/googletest/faq.html#CtorVsSetUp
.. _Don't Repeat Yourself: https://en.wikipedia.org/wiki/Don%27t_repeat_yourself

Reference:

- `Do setup teardown hurt test maintainability?`_

.. _Do setup teardown hurt test maintainability?: https://stackoverflow.com/questions/1087317/do-setup-teardown-hurt-test-maintainability

Writing maintainable unit tests
###############################

Avoid dependencies between test cases
-------------------------------------
If there are dependencies between test cases, the test cases may fail as the executing order changed, which
takes time to debug and hard to maintain.

There are several test "smell" that can hit test cases dependencies
 1. Constrained test order
     Tests expecting to be run in a specific order or expecting information from other test results
 2. Shared-state without rolling back
     Shared data between test cases, and a test finished without rolling back shared data.
     For example, a global mock variable shared between test cases doesn't verify and clear expectations after a test case finish,
     or setting an environment variable at a test beginning, but doesn't unset it after finishing.

Running test cases with random order is a quick and efficient method to check whether the test cases have dependencies or not.
A gtest user can refer to :ref:`Run unit tests with a random order`.

Avoid duplicate code between test cases
---------------------------------------

Avoid logic in test body
------------------------

Avoid many mocks in one test cases
----------------------------------

Avoid an abuse of integration tests
-----------------------------------
The integration tests are used to verify if the functionality meets requirements
with integration between software modules. They are considered as black-box tests.
Now we use robot framework to write integration tests according to test analysis
documents.

The unit tests are written by programmers which can reach deep into the code and
test independent units. They are considered as white-box tests.
Programmers use unit tests to verify a piece of code whenever they want. In addition,
unit tests can help programmers practice TDD.

Unit tests can run in any place, but integration tests could be highly dependent on
certain environment.

Although some checkpoints can be covered by integration tests as well as unit tests,
the integration tests run much slower. And for one single change, the number of unit
tests is usually more than integration tests. It has to take quite long time for
software verification if covering them all by robot cases.
Thus, it is necessary to limit the usage of robot cases only for integration testing
scope.

For example, to verify one functionality that operator uses a pair of user account
and password to login some remote server. Choose one typical pair of user account
and password is enough for integration tests. Multiple unit test cases are needed
to check whether the handling for different formats of user account and password
is expected.


Writing trustworthy unit tests
##############################

Test cases should run quickly
-----------------------------
The faster the tests, the more tests you can have in the suite and the more often you can run them.

With tests that run quickly, you can drastically shorten the feedback loop, to the
point where the tests begin to warn you about bugs as soon as you break the code, thus
reducing the cost of fixing those bugs almost to zero. On the other hand, slow tests
delay the feedback and potentially prolong the period during which the bugs remain
unnoticed, thus increasing the cost of fixing them. That's because slow tests discourage
you from running them often, and therefore lead to wasting more time moving in
a wrong direction.

The unit test cases should run as fast as it can, usually the time for one case should less than 1/10 of a second.

To reduce the case time, it is better:
    1. mock the system interfaces, especially for those time-consuming system functions, for example "sleep()".
    2. avoid use client server method to test your code, for example if your code is a server side code, you should
       not start a client to test it.
    3. should consider the testability when design your code, avoid use loop in your original code, for example,
       for Function A {while(true){code block}}, you should split the code block into a new function, and
       only do UT for the new function.

If it is really hard to reduce the test case time in some special cases(waiting events to continue, ...),
we suggest use integration tests or module tests to verify them.

Test cases should not include any bugs
--------------------------------------



Questions and Answers
#####################

How to mock for factory mode classes? 
-------------------------------------

How to write good integration tests with unit test framework? 
-------------------------------------------------------------

How to start unit testing of legacy code?  
-----------------------------------------

Unit testing of private methods?  
--------------------------------

Private methods are usually private for a good reason in the
developer's mind. Sometimes it's to hide implementation details,
so that the implementation can change later without the end
functionality changing.

When you test a private method, you are testing against a contract
internal to the system, which may well change. Internal contracts
are dynamic, and they can change when you re-factor the system.
When they change, your test could fail because some internal work
is being done differently, even though the overall functionality
of the system remains the same.

For testing purposes, the public contract (the overall functionality)
is all that you need to care about. Testing the functionality of
private methods may lead to breaking tests, even though the overall
functionality is correct.

Any private method is usually part of a bigger public method, so we
can add unit tests for these public methods to cover private methods.

With `TDD <https://web.yammer.com/main/threads/eyJfdHlwZSI6IlRocmVhZCIsImlkIjoiMTY1MzI5MzIwNjg3MjA2NCJ9>`_,
you usually write tests against methods that are public, and those 
public methods are later re-factored into calling smaller, private
methods, then you will not meet the question "how to add UT for
private method" when use TDD, so we suggest use TDD in your work.

How to write unit test for multi-threaded program?
--------------------------------------------------

You can verify the product logic of one single thread for
multi-threaded program in unit tests. The multi-threaded
aspects of the programs should not be tested in the unit test
directly.

There are race conditions(locks, synchronization, thread creating)
in multi-threaded program. Some race conditions may even be
difficult to be predicted.

The unit test focuses on the product logic. It is difficult to cover
these race conditions in unit testing. And the unit test will be
unstable due to some rare timing related race conditions.

The multi-threaded program is more suitable to be tested in module
and integration tests.

How could we do unit testing with network? 
------------------------------------------

Unit test should not communicate across network, networking transmission means your code under test interact with
external component(kernel stack) and over which you can not control.(Other examples are file system, threads, etc.)

Then we need to mock the networking APIs to make the code more testable.
For example, following codes used the API `ssize_t recv(int sockfd, void *buf, size_t len, int flags);`

.. code-block:: c++

    //MyClass.hpp
    namespace mynamespace
    {
        class MyClass
        {
        public:
            // Constructor & other methods..
            void doSomethingWithReceivedData(unsigned int seconds) noexcept
            {
                //some variable
                auto ret = ::recv(sockfd, buf, len, flags);
                if (ret > 0)
                {
                    //Do something
                }
            };

        };
    }


Direct usage for API `recv` makes it can not be mocked normally, then test the logic `//Do something` is harder.
Either we need to call real networking socket send/receive actions or we uses some global mock instances make the test
complex and hard to maintain.

Several ways can be used here to untangle the dependency, here is an example to use constructor injection.

.. code-block:: c++

    // NetworkingAPI.hpp
    //
    namespace mynamespace
    {
        class NetworkingAPI
        {
        public:
            // Constructor & other methods..
            virtual ssize_t recv(int sockfd, void *buf, size_t len, int flags);
            {
                return ::recv(sockfd, buf, len, flags);
            };
        };
    }

    //MyClass.hpp
    namespace mynamespace
    {
        class MyClass
        {
        public:
            // Other constructor/destructor & other methods..
            MyClass(NetworkingAPI& networkingApi) : networkingApi_(networkingApi);

            void doSomethingWithReceivedData() noexcept
            {
                //some variable
                auto ret = networkingApi_.recv(sockfd, buf, len, flags);
                if (ret > 0)
                {
                    //Do something
                }
            };
        private:
            NetworkingAPI &networkingApi_;
        };
    }

    //Unit test
    TEST(MyClassTest, IfReceivedSomethingThenShouldHappenSomeResult)
    {
        // MockNetworkingApi, if test fixture used then this can be put to SetUp()
        MockNetworkingApi mockNetworkingApi;
        int data[5] = {1,2,3,4,5};
        EXPECT_CALL(mockNetworkingApi, recv(_,NotNull(),_,_)).Times(1)
            .WillOnce(DoAll(setArrayArgument<1>(data, data + 5), Return(5*sizeof(int));

        auto myclass = std::make_unique<MyClass>(mockNetworkingApi);
        myclass->doSomethingWithReceivedData();
    }

By this way we can construct fake data and test the logic of `MyClass` easier.


How to add new features to legacy code with TDD? 
------------------------------------------------

How to mock c++ file system operation?
--------------------------------------

The simplest way to unit test file read/write operations is to separate file
opening/creation from reading/writing file content.

For example function for writing `Data` structure to given file can be separated
to two different functions:

.. code-block:: c++

    struct Data
    {
        /* ... */
    };

    void writeToStream(const Data& data, std::ostream& stream)
    {
        /* ... */
    }

    void writeToFile(const Data& data, const std::filesystem::path& path)
    {
        std::ofstream file(path);
        if (!file)
            throw std::system_error(errno, std::system_category(), path);
        writeToStream(data, file);
    }

The `writeToStream()` function concentrates on writing the given `Data` to the
given output stream (notice the `std::ostream` usage) and the `writeToFile()`
function concentrates on creating the file. This makes it possible to unit test
the `writeToStream()` function using `std::ostringstream`:

.. code-block:: c++

    TEST(ExampleTest, WriteToStreamSerializesTheGivenDataToTheGivenStream)
    {
        const Data data{/* ... */};
        std::ostringstream stream;
        writeToStream(data, stream);
        EXPECT_EQ("...", stream.str());
    }

Testing the happy path of the `writeToStream()` function can be skipped as it 
is so trivial. The error branch of the `writeToStream()` function can be unit
tested by using a dummy path:

.. code-block:: c++

    TEST(ExampleTest, IfCreatingFileFailsThenWriteToFileThrows)
    {
        EXPECT_THROW(writeToFile(Data{}, "/no/such/file/or/directory"), std::system_error);
    }

See :ref:`How to re-factor file parsing code to be testable` for real code
example for file reading.

If low-level functions, like `open(2)`, `close(2)`, `fstat(2)`, `read(2)`,
`write(2)`, etc. are needed, then they can be mocked like any other system calls.
See :ref:`Mock System calls`.

What UT framework is recommended for different programming languages in RCP?
----------------------------------------------------------------------------

- C++/C

  - Strongly recommend: `Gtest, Gmock`_
  - `CppUtest`_

.. _Gtest, Gmock: https://google.github.io/googletest/
.. _CppUtest: https://cpputest.github.io/

- Golang

  - `Testing`_
  - `Testify`_

.. _Testing: https://pkg.go.dev/testing
.. _Testify: https://github.com/stretchr/testify

- Python
  
  - `PyTest`_
  - `unittest`_
  
.. _PyTest: https://pypi.org/project/pytest/
.. _unittest:  https://docs.python.org/3/library/unittest.html

.. _How to re-factor file parsing code to be testable:

How to re-factor file parsing code to be testable?
--------------------------------------------------

In this section, we try to re-factor file parsing based on real piece of code
within project ipdp.

We could see the Reader function for example `Ipv4nodeStatReader` within IPDpCounter.cpp
doing more than one thing including file operations and content parsing.

.. code-block:: c++

    bool Ipv4nodeStatReader(Ipv4nodeCounter& ipv4nodeCounter)
    {
        bool retVal = true;
        char buf[BUFSIZE];
        int count = 1;
    #ifdef BITS64
        char format[] = "%*u %*u %*u %*u %*u %*u %*u %*u %*u %*u %*u %*u %lu %*u %*u %lu %*u %*u %*u";
    #else
        char format[] = "%*u %*u %*u %*u %*u %*u %*u %*u %*u %*u %*u %*u %llu %*u %*u %llu %*u %*u %*u";
    #endif

        std::string key;
    #ifdef TEST_ENABLE
        std::ifstream procFile("./ipdp/tst/rawdata/snmp");
    #else
        std::ifstream procFile("/proc/net/snmp");
    #endif
        char *pos, *id;

        int conv;
        try
        {
            while (!procFile.eof() && procFile.is_open())
            {
                procFile.getline(buf, BUFSIZE);
                int i = 0;
                while(buf[i]==' ')
                {
                    i++;
                }
                /*
                ...
                 skip some code about context parsing logic
                ...
                */
            }
        }
        catch (std::exception& e)
        {
            genapi_syslog(LOG_WARNING, "Exception %s in reading /proc/net/dev, offending line: %s\n", e.what(), buf);
            retVal = false;
        }

        if (procFile.is_open())
        {
            procFile.close();
        }

        return retVal;
    }

In order to test this function, `#ifdef TEST_ENABLE` macro is used to redirect file path in UT scenario
differ from real environment which makes unit test dependent on the file system
and requires to use actual file handling functions.

Better is to separate file operations and content parsing.
We use input stream as a parameter in content parsing function.
Note that the parameter is `std::istream` instead of `std::ifstream`.
Thus, we can pass `std::ifstream` object as input parameter in real code
and use `std::istringstream` in unit test to test the content parsing directly.

After doing this, the code is like:

.. code-block:: c++

    /*
        the content parsing function takes std::istream as input parameter and return counter object
        don't need #ifdef macro to redirect file path any more
    */
    Ipv4nodeCounter Ipv4nodeStatReader(std::istream& is)
    {
        Ipv4nodeCounter ipv4nodeCounter;
        char buf[BUFSIZE];
        int count = 1;
    #ifdef BITS64
        char format[] = "%*u %*u %*u %*u %*u %*u %*u %*u %*u %*u %*u %*u %lu %*u %*u %lu %*u %*u %*u";
    #else
        char format[] = "%*u %*u %*u %*u %*u %*u %*u %*u %*u %*u %*u %*u %llu %*u %*u %llu %*u %*u %*u";
    #endif

        std::string key;

        char *pos, *id;

        int conv;

        while (!is.eof())
        {
            is.getline(buf, BUFSIZE);
            int i = 0;
            while(buf[i]==' ')
            {
                i++;
            }
            /*
            ...
             skip some code about context parsing logic
            ...
            */
        }

        return ipv4nodeCounter;
    }

    /*
        wrap file operation and content parsing
        pass std::ifstream object as input parameter to content parsing function
    */
    Ipv4nodeCounter getIpv4nodeStateCounter(std::string& filePath) {
        std::ifstream procFile(filePath);
        if (!procFile.is_open())
        {
            throw std::runtime_error("File open failed, file path: " + filePath);
        }

        return Ipv4nodeStatReader(procFile);
    }

Let's have a look at the original unit test code.
We could not see any input data of this test case
since it redirects file path in unit test scenario as mentioned before.

.. code-block:: c++

    TEST(IPDpCounterTest, Ipv4nodeStatReaderSucceed)
    {
        bool ret = false;
        Ipv4nodeCounter ipv4nodeCounter;
        ret = Ipv4nodeStatReader(ipv4nodeCounter);
        EXPECT_TRUE(ret);
        EXPECT_EQ(ipv4nodeCounter.ReasmTimeout, 111);
        EXPECT_EQ(ipv4nodeCounter.ReasmFails, 222);
    }

But after re-factoring the code, we could use `std::istringstream` as input parameter to test content parsing directly.

.. code-block:: c++

    TEST(IPDpCounterTest, GetIpv4NodeCounterSuccess)
    {
        Ipv4nodeCounter ipv4nodeCounter;
        std::string inputString(R"(Ip: Forwarding DefaultTTL InReceives InHdrErrors InAddrErrors ForwDatagrams InUnknownProtos InDiscards InDelivers OutRequests OutDiscards OutNoRoutes ReasmTimeout ReasmReqds ReasmOKs ReasmFails FragOKs FragFails FragCreates
    Ip: 2 64 5449578 0 1 0 0 0 5386708 5334078 0 0 111 0 0 222 0 0 0
    Icmp: InMsgs InErrors InDestUnreachs InTimeExcds InParmProbs InSrcQuenchs InRedirects InEchos InEchoReps InTimestamps InTimestampReps InAddrMasks InAddrMaskReps OutMsgs OutErrors OutDestUnreachs OutTimeExcds OutParmProbs OutSrcQuenchs OutRedirects OutEchos OutEchoReps OutTimestamps OutTimestampReps OutAddrMasks OutAddrMaskReps
    Icmp: 8 0 8 0 0 0 0 0 0 0 0 0 0 8 0 8 0 0 0 0 0 0 0 0 0 0
    Tcp: RtoAlgorithm RtoMin RtoMax MaxConn ActiveOpens PassiveOpens AttemptFails EstabResets CurrEstab InSegs OutSegs RetransSegs InErrs OutRsts InCsumErrors
    Tcp: 1 200 120000 -1 42328 3557 152 4362 10 555 666 148921 48 20031 0
    Udp: InDatagrams NoPorts InErrors OutDatagrams RcvbufErrors SndbufErrors
    Udp: 554096 8 0 554104 0 0
    UdpLite: InDatagrams NoPorts InErrors OutDatagrams RcvbufErrors SndbufErrors
    UdpLite: 0 0 0 0 0 0)");
        std::istringstream isstr(inputString);
        // use istringstream as input data stream to directly test
        ipv4nodeCounter = Ipv4nodeStatReader(isstr);

        EXPECT_EQ(ipv4nodeCounter.ReasmTimeout, 111);
        EXPECT_EQ(ipv4nodeCounter.ReasmFails, 222);
    }

There is also existing good example for this topic,
ministarter has lot of code for parsing configuration from JSON files.

- There are two functions for parsing configuration [`1 <https://gitlabe1.ext.net.nokia.com/rcp-computing/ministarter/-/blob/pq1.4.3/src/daemon/readconfiguration.cpp#L258>`_].
- The first function is covered with three unit test cases [`2 <https://gitlabe1.ext.net.nokia.com/rcp-computing/ministarter/-/blob/pq1.4.3/tst/readconfiguration_test.cpp>`_].
- These three test cases access the real file system,
  but they concentrate only on successful and unsuccessful file I/O with pre-existing files [`3 <https://gitlabe1.ext.net.nokia.com/rcp-computing/ministarter/-/tree/pq1.4.3/tst/confdir>`_].
