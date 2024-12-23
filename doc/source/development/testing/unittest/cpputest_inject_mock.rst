How to inject a mock class with CppUTest at run-time
####################################################

This is a simple guide to teach you how to inject a mock class into product code 
and let your product class to be testable under CppUTest unit framework.

Note that this is same as way of gmock and gtest, you can see the video from
Turunen, Tuomo in the `yammer`_.

.. _yammer: https://web.yammer.com/main/org/nokia.com/threads/eyJfdHlwZSI6IlRocmVhZCIsImlkIjoiMTc3NTAzMzU2OTc2MzMyOCJ9

Below picture is about the relation between classes. ProductClassB depend on
ProductClassA(ProductClassB invoke the ProductClassA.method1() in 
ProductClassB.method()), now when we do unit test for ProductClassB, ProductClassA
should be mocked.

.. mermaid::

    classDiagram
    ProductClassA <|-- ProductClassAMock
    ProductClassA *-- ProductClassB
    ProductClassB_unittests --> ProductClassB
    ProductClassB_unittests --> ProductClassAMock
    Main --> ProductClassB
    class ProductClassA{
        +method1(arg1)
        +method2(arg1)
    }
    class ProductClassB{
        +ProductClassA& _ainstance;
        +method()
    }
    class ProductClassAMock{
        +method1(arg1)
        +method2(arg1)
    }
    class ProductClassB_unittests
    class Main


Below code shows how to use new way to mock a depended class by using CppUTest.

ProductClassA has some methods, these methods should be declared as virtual methods if
you want that these methods could be mocked.

ProductClassA.hpp:

.. code-block:: c++

    #ifndef _INCLUDE_PRODUCT_A_CLASS_
    #define _INCLUDE_PRODUCT_A_CLASS_
    #include <string>

    class ProductClassA {
        public:
            ProductClassA() {
            }

            ~ProductClassA() {
            }
        public:
            virtual int method1(const std::string& arg1);
            virtual int method2(const std::string& arg1);
    };

    #endif

ProductClassA.cpp:

.. code-block:: c++

    #include <iostream>
    #include "ProductClassA.hpp"

    int ProductClassA::method1(const std::string& arg1) {
        std::cout << arg1 << " ProductClassA class method1" <<std::endl;
        return 0;
    }

    int ProductClassA::method2(const std::string& arg1) {
        std::cout << arg1 << " ProductClassA class method2" <<std::endl;
        return 0;
    }

If you want to mock the depended class ProductClassA while unit testing 
ProductClassB, you should declare a private ProductClassA variable with 
reference type in ProductClassB, and you can have two constructors in
ProductClassB, one for product code and one for unit test code like below.

ProductClassB.hpp:

.. code-block:: c++

    #include <string>
    #include "ProductClassA.hpp"

    class ProductClassB {
        public:
            // for unit test
            ProductClassB(ProductClassA& ainstance): _ainstance(ainstance) {
            }
            // for product code
            ProductClassB(): _ainstance(ProductClassA()) {
            }
            ~ProductClassB() {
            }

        public:
            int method();
        
        private:
            ProductClassA& _ainstance;
    };

ProductClassB.cpp:

.. code-block:: c++

    #include "ProductClassB.hpp"
    #include <iostream>

    int ProductClassB::method() {
        std::cout<< "this is B class method"<<std::endl;
        return this->_ainstance.method1("B invoke");
    }

Class ProductClassAMock is derived class of ProductClassA, the methods will be re-implement with
mocking way.

ProductClassAMock.hpp:

.. code-block:: c++

    #include "ProductClassA.hpp"

    class ProductClassAMock : public ProductClassA
    {
    public:
        virtual int method1(const std::string& arg1);
        virtual int method2(const std::string& arg1);
    };

ProductClassAMock.cpp:

.. code-block:: c++

    #include "ProductClassAMock.hpp"

    #include <iostream>
    #include "CppUTest/TestHarness.h"
    #include "CppUTestExt/MockSupport.h"

    int ProductClassAMock::method1(const std::string& arg1) {
        std::cout << arg1 << " ProductClassAMock method1"<<std::endl;
        mock().actualCall("method1");
        return 0;
    }

    int ProductClassAMock::method2(const std::string& arg1) {
        mock().actualCall("method2");
        return 0;
    }


Unit test of ProductClassB will use the testing constructor to initialize
_ainstance with mock class ProductClassAMock in bInstance, then according
to C++ polymorphism mechanism, _ainstance.method1() is replaced with the
method1 from class ProductClassAMock.

ProductClassB_unittests.cpp:

.. code-block:: c++

    #include "ProductClassB.hpp"
    #include "ProductClassAMock.hpp"

    #include "CppUTest/CommandLineTestRunner.h"
    #include "CppUTest/TestHarness.h"
    #include "CppUTestExt/MockSupport.h"

    TEST_GROUP(BClassFooTests)
    {
        void teardown()
        {
            mock().clear();
        }
    };

    TEST(BClassFooTests, MockAsExpected)
    {
        mock().expectOneCall("method1");
        ProductClassAMock aMockInstance;
        ProductClassB bInstance(aMockInstance);
        bInstance.method();
        mock().checkExpectations();
    }

    int main(int ac, char** av)
    {
        return CommandLineTestRunner::RunAllTests(ac, av);
    }


Product code just use the constructor for product of ProductClassB, bInstance._ainstance will
be initialized with real class ProductClassA.

main.cpp:

.. code-block:: c++

    #include "ProductClassB.hpp"

    int main() {
        ProductClassB bInstance();
        bInstance.method();
        return 0;
    }


How to build the code:

.. code-block:: 

    # build main
    g++ ProductClassA.cpp ProductClassB.cpp main.cpp -o main

    # build unit test
    g++ ProductClassA.cpp ProductClassB.cpp ProductClassAMock.cpp ProductClassB_unittests.cpp -lstdc++ -lCppUTest -lCppUTestExt -o testrunner

Reference:

  - `CppUtest Mocking`_

.. _CppUtest Mocking: https://cpputest.github.io/mocking_manual.html

