******************************
Golang Unit Test Best Practice
******************************

.. contents:: Table of Contents
   :local:
   :depth: 1

Introduction
############

This document is to introduce the Golang unit test best practice. This is a
basic guide about how to write golang code to be testable, and some techniques
how to write golang unit tests.

About the common practice for unit test, please remember to refer `Unit Test Best Practice`_
guide.

.. _Unit Test Best Practice: https://rcp.gitlabe2-pages.ext.net.nokia.com/rcp-docs/development/testing/unittest/bestpractice.html#unit-tests-best-practice

Golang unit test frameworks
###########################

There are several unit test frameworks for Golang, here let's introduce some popular ones.

`Testing package`_
------------------

"testing" package is the standard library for testing in Go and provides a simple and 
lightweight API for writing unit tests.

.. _Testing package: https://golang.org/pkg/testing/

`Testify`_
----------

"testify" is widely used testing framework for Go that provides a rich set
of assertion functions, making it easy to write expressive and readable tests. Below are
main features for testify:

- Assertion functions
- Test suite and setup/teardown functions
- Mocking capabilities
- Helper functions
- Integration with other testing tools

.. _Testify: https://github.com/stretchr/testify

`Ginkgo`_
---------

"ginkgo" is a BDD-style testing framework for Go that allows you to write tests in a more
descriptive and readable manner. Below are main features for ginkgo:

- BDD-style syntax
- Nested describe and context blocks
- Focus and skip functions
- Custom matchers
- Asynchronous testing
- Parallel test execution
- BeforeEach and AfterEach functions
- Table drive testing
- Integration with other testing tools

.. _Ginkgo: https://onsi.github.io/ginkgo/

`Gomega`_
---------

"gomega" is a matcher library that can be used with "ginkgo" to make tests more expressive and readable.
Below are main features for gomega:

- Matcher functions like Equal, NotEqual, BeNil, ContainSubstring
- Custom matchers
- Matcher chaining
- Assertion failure messages
- Async support
- Integration with other testing tools
- Extensibility

.. _Gomega: https://onsi.github.io/gomega/

`GoConvey`_
-----------

GoConvey is another behavior-driven development (BDD)-style testing framework, that takes a somewhat
different approach to what a testing framework should do. Some key features are:

- Conforming to behavior-driven development (BDD)-style testing
- Using a domain-specific language (DSL)
- The ability to create self-documenting, highly readable tests
- Supports for contexts and scopes
- Support for assertions
- Availability of web UI

.. _GoConvey: https://github.com/smartystreets/goconvey

`Gomock`_
---------

Gomock is a mocking framework for Go that is used to create mock objects for testing. It is built on
top of the Go testing package and the Go interface system. Gomock makes it easy to create mock objects
and specify their behavior in tests. Interesting feature is about generate mock implementation from
Golang interfaces.

.. _Gomock: https://github.com/golang/mock

`Gocheck`_
----------

Gocheck includes features such as:

- Helpful error reporting to aid on figuring problems out (see below)
- Richer test helpers: assertions which interrupt the test immediately, deep multi-type comparisons, string matching, etc
- Suite-based grouping of tests
- Fixtures: per suite and/or per test set up and tear down
- Benchmarks integrated in the suite logic (with fixtures, etc)
- Management of temporary directories
- Panic-catching logic, with proper error reporting
- Proper counting of successes, failures, panics, missed tests, skips, etc
- Explicit test skipping
- Support for expected failures
- Verbosity flag which disables output caching (helpful to debug hanging tests, for instance)
- Multi-line string reporting for more comprehensible failures
- Inclusion of comments surrounding checks on failure reports
- Fully tested (it manages to test itself reliably)

.. _Gocheck: https://labix.org/gocheck

Golang interface
################

In Go, an interface is a collection of method signatures that define a set of
behaviors. An interface defines a contract that a type must fulfill in order
to be considered a valid implementation of the interface.

Interfaces are useful in Go because they allow you to write code that is
generic across different types. For example, you could define a function that
takes an Animal as a parameter and can be called with any type that implements
the Animal interface. This makes your code more flexible and reusable.

.. code-block:: golang

    type Animal interface {
        Speak() string
    }

    type Dog struct {
        Name string
    }

    func (d Dog) Speak() string {
        return d.Name + " says woof!"
    }

    type Cat struct {
        Name string
    }

    func (c Cat) Speak() string {
        return c.Name + " says meow!"
    }

    func main() {
        animals := []Animal{
            Dog{"Fido"},
            Cat{"Milo"},
        }
        for _, animal := range animals {
            fmt.Println(animal.Speak())
        }
    }

Golang interface use cases
##########################

- Abstraction: Interfaces can be used to define an abstraction for a set of
  related types, without knowing their specific implementation details. This
  can be useful in situations where you want to write code that is generic
  across multiple types.
- Dependency injection: Interfaces can be used for dependency injection,
  which is a design pattern that allows you to inject dependencies (such as a
  database connection or a logger) into your code at runtime. By defining an
  interface for each dependency, you can easily swap out the implementation of
  that dependency without changing the rest of your code.
- Testing: Interfaces can be used to write unit tests for your code. By
  defining interfaces for your dependencies, you can easily create mock
  implementations of those dependencies for testing purposes.
- API design: Interfaces can be used to define the API of a package or library.
  By defining a set of interfaces that your users can implement, you can give
  them more control over the behavior of your code.
- Polymorphism: Interfaces can be used to achieve polymorphism in Go.
  Polymorphism allows you to write code that can work with objects of
  different types, as long as those objects implement a common interface.

Testify mock framework
######################

Testify is a popular testing framework for Go that provides a variety of utilities
for writing and running tests. One of its features is a mock framework that allows
you to easily create and use mock objects in your tests.

To use Testify's mock framework, you first need to import the
github.com/stretchr/testify/mock package.

.. code-block:: golang

    package main

    import (
        "errors"
        "testing"

        "github.com/stretchr/testify/assert"
        "github.com/stretchr/testify/mock"
    )

    type Service interface {
        GetResult() (string, error)
    }

    type ServiceMock struct {
        mock.Mock
    }

    func (m *ServiceMock) GetResult() (string, error) {
        args := m.Called()
        return args.String(0), args.Error(1)
    }

    func DoSomething(s Service) (string, error) {
        result, err := s.GetResult()
        if err != nil {
            return "", err
        }
        return "Result: " + result, nil
    }

    func TestDoSomethingSuccess(t *testing.T) {
        mockService := new(ServiceMock)
        mockService.On("GetResult").Return("success", nil)

        result, err := DoSomething(mockService)
        assert.NoError(t, err)
        assert.Equal(t, "Result: success", result)

        mockService.AssertExpectations(t)
    }

    func TestDoSomethingError(t *testing.T) {
        mockService := new(ServiceMock)
        mockService.On("GetResult").Return("", errors.New("failed"))

        result, err := DoSomething(mockService)
        assert.Error(t, err)
        assert.Equal(t, "", result)

        mockService.AssertExpectations(t)
    }


Generate mock implementation from Golang interface via mockery
##############################################################

`Mockery`_ is a code generator that creates mock implementations of Golang
interfaces automatically. The mocks generated by this tool are based on the
github.com/stretchr/testify/mock packages.

Prerequisites
-------------

- Install via Go

.. code-block:: bash

    go install github.com/vektra/mockery/v2@latest

- Install via docker

.. code-block:: bash

    docker pull vektra/mockery
    docker run -v "$PWD":/src -w /src vektra/mockery --all

Configuration
-------------

Usually, mockery command has a lot of options to configure the mock generation. You can
use --help to get help, and also you can use mockery.yaml to configure the mock generation.

Here is a simple example:

.. code-block:: bash

    $ cat .mockery.yaml
    inpackage: True
    with-expecter: False
    inpackage-suffix: True
    testonly: False

For more details about the configuration, please refer to `Mockery`_.

Generate mock implementation
----------------------------

Here is a simple example provided:

.. code-block:: golang

    // code in string.go
    package example_project

    //go:generate mockery --name Stringer
    type Stringer interface {
        String() string
        GetMyName() string
    }

You can use this steps to test:

.. code-block:: bash

    mkdir example_project
    cd example_project
    # Copy the code to string.go
    # Copy above .mockery.yaml to current directory
    mockery --all
    ls -la
    -rw-r--r-- 1    80 May 22 08:46 .mockery.yaml
    -rw-r--r-- 1  1179 Jun  8 10:35 Stringer_mock.go
    -rw-r--r-- 1   135 May 22 08:44 string.go

Mockery will generate a mock implementation for the Stringer interface in the file Stringer_mock.go:

.. code-block:: golang

    // Code generated by mockery v2.27.1. DO NOT EDIT.

    package example_project

    import mock "github.com/stretchr/testify/mock"

    // MockStringer is an autogenerated mock type for the Stringer type
    type MockStringer struct {
            mock.Mock
    }

    // GetMyName provides a mock function with given fields:
    func (_m *MockStringer) GetMyName() string {
            ret := _m.Called()

            var r0 string
            if rf, ok := ret.Get(0).(func() string); ok {
                    r0 = rf()
            } else {
                    r0 = ret.Get(0).(string)
            }

            return r0
    }

    // String provides a mock function with given fields:
    func (_m *MockStringer) String() string {
            ret := _m.Called()

            var r0 string
            if rf, ok := ret.Get(0).(func() string); ok {
                    r0 = rf()
            } else {
                    r0 = ret.Get(0).(string)
            }

            return r0
    }

    type mockConstructorTestingTNewMockStringer interface {
            mock.TestingT
            Cleanup(func())
    }

    // NewMockStringer creates a new instance of MockStringer. It also registers a testing interface on the mock and a cleanup function to assert the mocks expectations.
    func NewMockStringer(t mockConstructorTestingTNewMockStringer) *MockStringer {
            mock := &MockStringer{}
            mock.Mock.Test(t)

            t.Cleanup(func() { mock.AssertExpectations(t) })

            return mock
    }

.. _Mockery: https://vektra.github.io/mockery/latest/


Provide mock implementation for your interfaces
###############################################

Good recommendation is that when you do abstraction for some behaviors to be
Golang interfaces, after you finish the implementation of such interface, you
can also provide the mock implementation at same time.

Via this way the good aspect is that interface users can easily test their code
and more willing to write testable code.


How to make Golang code to be testable?
#######################################

A simple example
----------------

.. code-block:: golang

    //Code in app.go
    package main

    import "fmt"

    type DB interface {
        Get(key string) (string, error)
        Set(key, value string) error
    }

    type App struct {
        db DB
    }

    func NewApp(db DB) *App {
        return &App{db: db}
    }

    func (a *App) GetAndPrint(key string) error {
        value, err := a.db.Get(key)
        if err != nil {
            return err
        }
        fmt.Println(value)
        return nil
    }

    func (a *App) Set(key, value string) error {
        return a.db.Set(key, value)
    }

In this example, we have defined a DB interface that defines the methods Get and Set.
We then define an App struct that has a db field of type DB. We also define a NewApp
function that creates a new App instance with the specified DB implementation.

The GetAndPrint method on the App struct calls the Get method on the db field and
prints the returned value to the console. The Set method simply calls the Set method
on the db field.

By defining the DB interface and using it as the type for the db field in the App
struct, we have made our code more testable. We can easily create a mock implementation
of the DB interface for testing purposes and pass it to the NewApp function to create a
new App instance with the mock implementation.

Here's an example of how to test the App struct using a mock implementation of the DB
interface:

.. code-block:: golang

    //Unit tests for APP.
    package main

    import (
        "errors"
        "testing"

        "github.com/stretchr/testify/assert"
        "github.com/stretchr/testify/mock"
    )

    type MockDB struct {
        mock.Mock
    }

    func (m *MockDB) Get(key string) (string, error) {
        args := m.Called(key)
        return args.String(0), args.Error(1)
    }

    func (m *MockDB) Set(key, value string) error {
        args := m.Called(key, value)
        return args.Error(0)
    }

    func TestGetAndPrintSuccess(t *testing.T) {
        mockDB := new(MockDB)
        mockDB.On("Get", "foo").Return("bar", nil)

        app := NewApp(mockDB)

        err := app.GetAndPrint("foo")
        assert.NoError(t, err)

        mockDB.AssertExpectations(t)
    }

    func TestGetAndPrintError(t *testing.T) {
        mockDB := new(MockDB)
        mockDB.On("Get", "foo").Return("", errors.New("failed"))

        app := NewApp(mockDB)

        err := app.GetAndPrint("foo")
        assert.Error(t, err)

        mockDB.AssertExpectations(t)
    }

In this test, we create a new MockDB object and define its behavior using the
On method. We then create a new App instance with the mock implementation and
call the GetAndPrint method with a key. We check the error value returned by
the method and use the AssertExpectations method on the mock object to verify
that all expected method calls were made during the test.

Example from RCP project
------------------------

Precondition: use the Golang UT framework proposed here `What UT framework is 
recommended for different programming languages in RCP?`_.
The Golang is a little different from C++, lots of API functions are provided as 
a procedure programming language(regular function).In Go the idea is that the 
interfaces would not be provided by the library authors, but that users of 
libraries would build interfaces that suit their use case. This is to avoid 
so-called interface hell. We can use the "interface" feature to make the 
source code easy to mock for UT. It needs to change our mindset when providing 
Golang APIs, we need to provide the APIs as "interface" instead of "function" 
as much as possible.
We have two options to achieve such UT mocking.

- 1. using a global variable to hold the interface object and assign 
  it in UT code. In the product code, the global variable "**OSUtils**" is 
  used to call the os.ReadDir().
  It is possible to assign **OSUtils** with a mock object.

  .. code-block:: golang

        package product_code
        type OS interface{
            ReadDir(string) []fs.DirEntry, nil
        }
        Type OSUtils struct{
        }
        Func (os *OSUtils) ReadDir(path string) []fs.DirEntry, nil
        {
            return os.ReadDir(path)
        }

        Var Os OSUtils = &OSUtils{}

        func cleanupWorkspace(conf config.Config) error {
            entries, err := Os.ReadDir(conf.Workspace)
            if err != nil {
                return nil
            }
            if noNeedToCleanupFolder(conf.Workspace, entries) {
                return nil
            }
        }

- 2. inject the mock object by the NewFunction(), the source code
  provides the service object by the NewFunction(). It is not recommended to
  provide the service API by function directly. The NewCertMan(params
  ...interface{}) is using variadic parameter as input argument, The
  NewCertMan() is used in product code, and the NewCertMan(mockObj...) can be
  used in UT code.

  The CertMan source code can be tested with any required mock object, this
  option is quite like C++ class constructor injection.

  .. code-block:: golang

        type CertManager interface {
            GetCertList() ([]models.Cert, error)
        }

        type CertMan struct {
            cert  models.CertDBProcessor
            util  certutils.CertParser
            cList models.CertsDBProcessor
        }

        func NewCertMan(params ...interface{}) *CertMan {
            cm := CertMan{cert: new(models.Cert), util: certutils.NewCertParser(), cList: models.NewCertsDBProcessor()}
            for _, val := range params {
                switch obj := val.(type) {
                case models.CertDBProcessor:
                    cm.cert = obj
                case certutils.CertParser:
                    cm.util = obj
                case models.CertsDBProcessor:
                    cm.cList = obj
                default:
                    ...
                }
            }
            return &cm
        }

        func (cm *CertMan) GetCertList() ([]models.Cert, error) {
            return cm.cList.GetCertList()
        }

  The UT code can be like this, you can define the mock type, and control it 
  in UT code. This UT sample is suitable for both above options:

  .. code-block:: golang

        // define the mock
        type cListMock struct {
            mock.Mock
        }
        func (clm *cListMock) GetCerts(filter models.CertFilter) ([]models.Cert, error) {
            args := clm.Called()
            return []models.Cert{}, args.Error(1)
        }

        // UT code
        func TestGetCertsFailedWithReturnEmpty(t *testing.T) {
            ......
            expectedError := errors.New("GetCertsFailed")
            cListMock := new(cListMock)
            cListMock.On("GetCerts", filter).Return([]models.Cert{}, expectedError)
            certManObj := app.NewCertMan(cListMock)  // inject the mock object

            outputCerts, err := certManObj.GetCerts(models.CertFilter{}) // test the target function.

            assert.Equal(t, expectedError, err)
            assert.Empty(t, outputCerts)
            cListMock.AssertExpectations(t)
            ......
        }

.. _What UT framework is recommended for different programming languages in RCP?: https://rcp.gitlabe2-pages.ext.net.nokia.com/rcp-docs/development/testing/unittest/bestpractice.html#what-ut-framework-is-recommended-for-different-programming-languages-in-rcp

How to mock OS methods
----------------------

Define the wrapper interface, here use OS method stat as a example:

.. code-block:: golang

    type OsWrapper interface {
        Stat(name string) (fs.FileInfo, error)
    }

Implement interface with wrapper method:

.. code-block:: golang

    type OsWrapperImpl struct {
    }

    func NewOsUtil() *OsWrapperImpl {
	    return &OsWrapperImpl{}
    }

    func (o *OsWrapperImpl) Stat(name string) (fs.FileInfo, error) {
        return os.Stat(name)
    }

Implement interface with mock:

.. code-block:: golang

    import (
        "io/fs"

        "github.com/stretchr/testify/mock"
    )

    type OsAdapterMock struct {
        mock.Mock
    }

    func (o *OsAdapterMock) Stat(name string) (fs.FileInfo, error) {
        args := o.Called(name)
        return nil, args.Error(1)
    }

    func SetupOsUtilMock() *OsAdapterMock {
        m := new(OsAdapterMock)
        return m
    }

Table driven unit testing
#########################

Table-driven tests with sub tests can be a helpful pattern for writing tests to avoid
duplicating code when the core test logic is repetitive.

If a system under test needs to be tested against multiple conditions where certain
parts of the the inputs and outputs change, a table-driven test should be used to reduce
redundancy and improve readability.

Bad example:

.. code-block:: golang

    // func TestSplitHostPort(t *testing.T)

    host, port, err := net.SplitHostPort("192.0.2.0:8000")
    require.NoError(t, err)
    assert.Equal(t, "192.0.2.0", host)
    assert.Equal(t, "8000", port)

    host, port, err = net.SplitHostPort("192.0.2.0:http")
    require.NoError(t, err)
    assert.Equal(t, "192.0.2.0", host)
    assert.Equal(t, "http", port)

    host, port, err = net.SplitHostPort(":8000")
    require.NoError(t, err)
    assert.Equal(t, "", host)
    assert.Equal(t, "8000", port)

    host, port, err = net.SplitHostPort("1:8")
    require.NoError(t, err)
    assert.Equal(t, "1", host)
    assert.Equal(t, "8", port)

Good example:

.. code-block:: golang

    // func TestSplitHostPort(t *testing.T)

    tests := []struct{
    give     string
    wantHost string
    wantPort string
    }{
    {
        give:     "192.0.2.0:8000",
        wantHost: "192.0.2.0",
        wantPort: "8000",
    },
    {
        give:     "192.0.2.0:http",
        wantHost: "192.0.2.0",
        wantPort: "http",
    },
    {
        give:     ":8000",
        wantHost: "",
        wantPort: "8000",
    },
    {
        give:     "1:8",
        wantHost: "1",
        wantPort: "8",
    },
    }

    for _, tt := range tests {
    t.Run(tt.give, func(t *testing.T) {
        host, port, err := net.SplitHostPort(tt.give)
        require.NoError(t, err)
        assert.Equal(t, tt.wantHost, host)
        assert.Equal(t, tt.wantPort, port)
    })
    }

Test tables make it easier to add context to error messages, reduce duplicate logic,
and add new test cases.

We follow the convention that the slice of structs is referred to as tests and
each test case test. Further, we encourage to differentiate the input and output
values for each test case with **give** and **want** prefixes.

.. code-block:: golang

    tests := []struct{
    give     string
    wantHost string
    wantPort string
    }{
    // ...
    }

    for _, test := range tests {
    // ...
    }

Avoid Unnecessary Complexity in Table Tests
-------------------------------------------

Table tests can be difficult to read and maintain if the sub tests contain conditional
assertions or other branching logic. Table tests should NOT be used whenever there needs
to be complex or conditional logic inside sub tests (i.e. complex logic inside the for loop).

Large, complex table tests harm readability and maintainability because test readers may
have difficulty debugging test failures that occur.

Table tests like this should be split into either multiple test tables or multiple
individual Test functions.

Some ideals to aim for are:

- Focus on the narrowest unit of behavior.
- Minimize "test depth", and avoid conditional assertions (see below).
- Ensure that all table fields are used in all tests.
- Ensure that all test logic runs for all table cases.

In this context, "test depth" means "within a given test, the number of successive
assertions that require previous assertions to hold" (similar to cyclomatic complexity).
Having "shallower" tests means that there are fewer relationships between assertions
and, more importantly, that those assertions are less likely to be conditional by default.

Concretely, table tests can become confusing and difficult to read if they use multiple
branching pathways (e.g. shouldError, expectCall, etc.), use many if statements for specific
mock expectations (e.g. shouldCallFoo), or place functions inside the table
(e.g. setupMocks func(\*FooMock)).

However, when testing behavior that only changes based on changed input, it may be preferable
to group similar cases together in a table test to better illustrate how behavior changes
across all inputs, rather than splitting otherwise comparable units into separate tests and
making them harder to compare and contrast.

If the test body is short and straightforward, it's acceptable to have a single branching
pathway for success versus failure cases with a table field like shouldError to specify error
expectations.

Bad example:

.. code-block:: golang

    func TestComplicatedTable(t *testing.T) {
    tests := []struct {
        give          string
        want          string
        wantErr       error
        shouldCallX   bool
        shouldCallY   bool
        giveXResponse string
        giveXErr      error
        giveYResponse string
        giveYErr      error
    }{
        // ...
    }

    for _, tt := range tests {
        t.Run(tt.give, func(t *testing.T) {
        // setup mocks
        ctrl := gomock.NewController(t)
        xMock := xmock.NewMockX(ctrl)
        if tt.shouldCallX {
            xMock.EXPECT().Call().Return(
            tt.giveXResponse, tt.giveXErr,
            )
        }
        yMock := ymock.NewMockY(ctrl)
        if tt.shouldCallY {
            yMock.EXPECT().Call().Return(
            tt.giveYResponse, tt.giveYErr,
            )
        }

        got, err := DoComplexThing(tt.give, xMock, yMock)

        // verify results
        if tt.wantErr != nil {
            require.EqualError(t, err, tt.wantErr)
            return
        }
        require.NoError(t, err)
        assert.Equal(t, want, got)
        })
    }
    }

Good example:

.. code-block:: golang

    func TestShouldCallX(t *testing.T) {
    // setup mocks
    ctrl := gomock.NewController(t)
    xMock := xmock.NewMockX(ctrl)
    xMock.EXPECT().Call().Return("XResponse", nil)

    yMock := ymock.NewMockY(ctrl)

    got, err := DoComplexThing("inputX", xMock, yMock)

    require.NoError(t, err)
    assert.Equal(t, "want", got)
    }

    func TestShouldCallYAndFail(t *testing.T) {
    // setup mocks
    ctrl := gomock.NewController(t)
    xMock := xmock.NewMockX(ctrl)

    yMock := ymock.NewMockY(ctrl)
    yMock.EXPECT().Call().Return("YResponse", nil)

    _, err := DoComplexThing("inputY", xMock, yMock)
    assert.EqualError(t, err, "Y failed")
    }

This complexity makes it more difficult to change, understand, and prove the
correctness of the test.

While there are no strict guidelines, readability and maintainability should
always be top-of-mind when deciding between Table Tests versus separate tests
for multiple inputs/outputs to a system.
