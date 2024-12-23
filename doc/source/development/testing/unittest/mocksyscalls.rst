.. _Mock System calls:

Mock System calls
#################

If a function uses some system call in it, for example sleep(n).
The unit test case will run very slow if real `sleep` is invoked.

The real system call does not need to be invoked in unit test case.

In order to avoid using system calls directly wrapping these Unix
standard APIs in a class so that they can be mockable. Then inject
it into target class and using mock class instead of real system call
in unit test.

Please also refer to chapter `3.4.3 Inject a fake at the constructor level <https://livebook.manning.com/book/the-art-of-unit-testing-second-edition/chapter-3/85>`_
in `the art of unit testing second edition` for more details.

For example:

.. code-block:: c++

    //system.hpp
    namespace mynamespace
    {
        class System
        {
        public:
            System() = default;
            virtual ~System() = default;

            System(const System&) = delete;
            System& operator=(const System&) = delete;
            System(System&&) = delete;
            System& operator=(System&&) = delete;

            virtual unsigned int sleep(unsigned int seconds) noexcept;

            static System& getSystem() noexcept;
        };
    }

    //system.cpp
    #include "system.hpp"
    #include <unistd.h>
    unsigned int System::sleep(unsigned int seconds)
    {
        return ::sleep(seconds);
    }

    System& System::getSystem() noexcept
    {
        static System system;
        return system;
    }

    //ClassA.hpp
    #include "system.hpp"
    namespace classA
    {
        class ClassA
        {
          public:
            ClassA();
            //Add a new constructor
            ClassA(mynamespace::System& system);
            ...
           private:
             mynamespace::System& system;
             ...
        }
    }

    //ClassA.cpp
    //Using real system call in practical usage
    ClassA::ClassA():
      ClassA(System::getSystem())
    {
    }

    ClassA::ClassA(System& system):
        system(system)
    {
    }

    ClassA::functionA():
    {
        ...
        int sleep_time = 60;
        system.sleep(sleep_time);
        ...
    }

    //unit test
    namespace
    {
      class ClassATest: public testing::Test
      {
      public:
          StrictMock<SystemMock> systemMock;
          ClassATest()
          {
              classA = std::make_shared<ClassA>(systemMock);
          }

          void expectSleep()
          {
              EXPECT_CALL(systemMock, sleep(Eq(60)))
                  .Times(1)
                  .WillOnce(Return(0));
          }
       };
    }

Example links:

`system.hpp <https://gerrite1.ext.net.nokia.com/gitweb?p=scm_rcp/SS_CertMan.git;a=blob;f=common/include/system.hpp;h=51517320585eefad8ea56ec8756e3c1cc5b88e13;hb=6fee9ec48ff7184583d3e8e113fcf7fd8c6f4f1b>`_

`system.cpp <https://gerrite1.ext.net.nokia.com/gitweb?p=scm_rcp/SS_CertMan.git;a=blob;f=common/src/system.cpp;h=7473d4e07fba2e897b0c3769162fc2101ee40502;hb=6fee9ec48ff7184583d3e8e113fcf7fd8c6f4f1b>`_

`autohelper.hpp <https://gerrite1.ext.net.nokia.com/gitweb?p=scm_rcp/SS_CertMan.git;a=blob;f=include/autokurhelper.hpp;h=bb6c9e299be051b363b76b8436710c7c34bbb541;hb=6fee9ec48ff7184583d3e8e113fcf7fd8c6f4f1b>`_

`autokurhelper.cpp <https://gerrite1.ext.net.nokia.com/gitweb?p=scm_rcp/SS_CertMan.git;a=blob;f=autocmp/autokurhelper.cpp;h=0291d3362f33df0386084567cc1e2610ba3ff6fa;hb=6fee9ec48ff7184583d3e8e113fcf7fd8c6f4f1b>`_

`autokurhelper_test.cpp <https://gerrite1.ext.net.nokia.com/gitweb?p=scm_rcp/SS_CertMan.git;a=blob;f=autocmp/tst/autokurhelper_test.cpp;h=88f636ae7fdd5fcd3cb08126e96390e5a0984538;hb=6fee9ec48ff7184583d3e8e113fcf7fd8c6f4f1b>`_
