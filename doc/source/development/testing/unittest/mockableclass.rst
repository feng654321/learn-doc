Mockable class
##############

One usual problem with mocking external dependencies is that
if external interface class is mocked and the external project
adds pure virtual interface, then the unit tests mocking such
class will fail to compile.

One popular method to avoid this kind of problems is to provide
"mockable" class on top of the real class in the public interface.
See concrete example from `SDL mockableasyncstorage.hpp`_.

.. _SDL mockableasyncstorage.hpp: https://gitlabe1.ext.net.nokia.com/rcpsdl/shareddatalayer/-/blob/pq3.15.2/include/sdl/tst/mockableasyncstorage.hpp

.. code-block:: c++

        // File: my/tst/mockablemyclass.hpp

        namespace myclass
        {
            namespace tst
            {
                /**
                 * @brief Helper class for mocking MyClass
                 *
                 * MockableMyClass can be used in application's unit tests to mock MyClass.
                 * MockableMyClass guarantees that even if new functions are added into
                 * MyClass, existing unit test cases do not have to be modified unless
                 * those new functions are used.
                 *
                 * If a function is not mocked but called in unit tests, then the call
                 * will be logged and the process will abort.
                 */
                class MockableMyClass: public MyClass
                {
                public:
                    MockableMyClass() {}

                    virtual int myMethod() const override { logAndAbort(__PRETTY_FUNCTION__); }

                private:
                    static void logAndAbort(const char* function) noexcept __attribute__((__noreturn__))
                    {
                        std::cerr << "MockableMyClass: calling not-mocked function "
                                  << function << ", aborting" << std::endl;
                        abort();
                    }
                };
            }
        }


How to use it?
--------------

1. Application derives its own unit tests mock from **MockableMyClass**
   instead of **MyClass**
2. The **MockableMyClass** is used only in unit tests, never in
   production code

The benefits?
-------------

1. **MockableMyClass** doesn't have pure virtual functions, so adding
   new public pure virtual functions to the real **MyClass** doesn't
   require any changes in application's mock
2. **MockableMyClass** does not depend on any particular unit test
   framework, so application can freely select the unit test framework
   they want to use (**gmock** or something else)

The subsystem providing such "mockable" class naturally must make sure
that whenever new pure virtual functions are added to the real class,
also the "mockable" class is updated. SDL does this with simple
`mockable unit test`_.

.. _mockable unit test: https://gitlabe1.ext.net.nokia.com/rcpsdl/shareddatalayer/blob/master/tst/mockableasyncstorage_test.cpp

.. code-block:: c++

        #include <my/tst/mockablemyclass.hpp>
        #include <gtest/gtest.h>

        TEST(MockableMyClassTest, CanCreateInstance)
        {
            shareddatalayer::tst::MockableMyClass mockableMyClass;
            static_cast<void>(mockableMyClass);
        }

