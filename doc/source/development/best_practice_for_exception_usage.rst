*****************************************
Best practices for exception usage in C++
*****************************************

.. contents:: :local:

Purpose of using exception
##########################

Exception handling is a modern C++ way of handling errors. Reasonable use of
exceptions can make the code more robust, scalable, and maintainable, and make
the code more like a clean code. At the same time, the wrong use of exceptions
will cause unexpected errors. Therefore, this article provides the following
programming suggestions to facilitate understanding the use of exceptions.

The principles of using exception
#################################

Use exceptions for error handling only
**************************************

**Reason**

To keep error handling separated from "ordinary code" C++ implementations tend
to be optimized based on the assumption that exceptions are rare.

**Bad example**

.. code-block:: c++

    // don't: exception not used for error handling
    int find_index(vector<string>& vec, const string& x)
    {
        try {
            for (size_t i = 0; i < vec.size(); ++i)
                if (vec[i] == x) throw i;  // found x
        }
        catch (int i) {
            return i;
        }
        return -1;   // not found
    }


Perhaps, people would use exception to do some non error handling scenarios.
Like above example, this is more complicated and most likely runs much slower
than the obvious alternative. There is nothing exceptional about finding a value
in a vector. Directly return the index would be more effective.

Use asserts to check for errors that should never occur
*******************************************************

Use asserts to check for errors that should never occur, Use exceptions to check
for errors that might occur, for example, errors in input validation on
parameters of public functions. For more information, see the `Exceptions versus
assertions`_ section.

Design your exception-handling strategy around invariants
*********************************************************

**Reason**

To use an object it must be in a valid state (defined formally or informally by
an invariant) and to recover from an error every object not destroyed must be in
a valid state.

**Note** An invariant is a logical condition for the members of an object that a
constructor must establish for the public member functions to assume. For the
definition of invariant, please refer to: `Class invariant`_.

.. _Class invariant: <https://www.geeksforgeeks.org/what-is-class-invariant/>

Use RAII to prevent leaks while using exception
***********************************************

**Reason**

Leaks are typically unacceptable. Manual resource release is error-prone.
RAII(Resource Acquisition Is Initialization) is the simplest, most systematic
way of preventing leaks.

**Example**

.. code-block:: c++

    void f1(int i)   // Bad: possible leak
    {
        int* p = new int[12];
        // ...
        if (i < 17) throw Bad{"in f()", i};
        // ...
    }

We could carefully release the resource before the throw:

.. code-block:: c++

    void f2(int i)   // Clumsy and error-prone: explicit release
    {
        int* p = new int[12];
        // ...
        if (i < 17) {
            delete[] p;
            throw Bad{"in f()", i};
        }
        // ...
    }
    // This is verbose. In larger code with multiple possible throws explicit
    // releases become repetitive and error-prone.

    void f3(int i)   // OK: resource management done by a handle (but see below)
    {
        auto p = make_unique<int[]>(12);
        // ...
        if (i < 17) throw Bad{"in f()", i};
        // ...
    }
    // Note that this works even when the throw is implicit because it happened
    // in a called function:

    void f4(int i)   // OK: resource management done by a handle (but see below)
    {
        auto p = make_unique<int[]>(12);
        // ...
        helper(i);   // might throw
        // ...
    }
    // Unless you really need pointer semantics, use a local resource object:

    void f5(int i)   // OK: resource management done by local object
    {
        vector<int> v(12);
        // ...
        helper(i);   // might throw
        // ...
    }
    // That’s even simpler and safer, and often more efficient.

When exceptions cannot be used, simulate RAII. That is, systematically check
that objects are valid after construction and still release all resources in the
destructor. One strategy is to add a valid() operation to every resource handle:

.. code-block:: c++

    void f()
    {
        vector<string> vs(100);   // not std::vector: valid() added
        if (!vs.valid()) {
            // handle error or exit
        }

        ifstream fs("foo");   // not std::ifstream: valid() added
        if (!fs.valid()) {
            // handle error or exit
        }

        // ...
    } // destructors clean up as usual

Use noexcept when a function does not throw a exception
*******************************************************

**Reason**

For compatibility history reasons, C++ functions can throw exceptions by
default. To make error handling systematic, robust, and efficient, The noexcept
flag should be explicitly added to functions that do not need to throw
exceptions.

**Example**

.. code-block:: c++

    double compute(double d) noexcept
    {
        return log(sqrt(d <= 0 ? 1 : d));
    }

Here, we know that compute will not throw because it is composed out of
operations that don't throw. By declaring compute to be noexcept, we give the
compiler and human readers information that can make it easier for them to
understand and manipulate compute.

**Note**

Many standard-library functions are noexcept including all the standard-library
functions "inherited" from the C Standard Library.

**Example**

.. code-block:: c++

    vector<double> munge(const vector<double>& v) noexcept
    {
        vector<double> v2(v.size());
        // ... do something ...
    }
    
The noexcept here states that I am not willing or able to handle the situation
where I cannot construct the local vector. That is, I consider memory exhaustion
a serious design error (on par with hardware failures) so that I'm willing to
crash the program if it happens.

**Note**

Do not use traditional exception-specifications.

Never throw while being the direct owner of an object
*****************************************************

**Reason**

That would be a leak.

**Example**

.. code-block:: c++

    void leak(int x)   // don't: might leak
    {
        auto p = new int{7}; // here, 'p' is the direct owner of an object,
        // only function leak can free its resource
        if (x < 0) throw Get_me_out_of_here{};  // might leak *p
        // ...
        delete p;   // we might never get here
    }
    // One way of avoiding such problems is to use resource handles consistently:

    void no_leak(int x)
    {
        auto p = make_unique<int>(7);
        if (x < 0) throw Get_me_out_of_here{};  // will delete *p if necessary
        // ...
        // no need for delete p
    }
    // Another solution (often better) would be to use a local variable to
    // eliminate explicit use of pointers:

    void no_leak_simplified(int x)
    {
        vector<int> v(7);
        // ...
    }

**Note**

If you have a local "thing" that requires cleanup, but is not represented by
an object with a destructor, such cleanup must also be done before a throw.

Use purpose-designed user-defined types as exceptions (not built-in types)
**************************************************************************

**Reason**

A user-defined type can better transmit information about an error to a handler.
Information can be encoded into the type itself and the type is unlikely to
clash with other people's exceptions.

**Example**

.. code-block:: c++

    throw 7; // bad

    throw "something bad";  // bad

    throw std::exception{}; // bad - no info
    
    // Deriving from std::exception gives the flexibility to catch the specific
    // exception or handle generally through std::exception:

    class MyException : public std::runtime_error
    {
    public:
        MyException(const string& msg) explicit: std::runtime_error{msg} {}
        // ...
    };

    // ...

    throw MyException{"something bad"};  // good

    class MyCustomError final {};  // not derived from std::exception

    // ...

    throw MyCustomError{};  // good - handlers must catch this type (or ...)
    
    // Library types derived from std::exception can be used as generic exceptions
    // if no useful information can be added at the point of detection:

    throw std::runtime_error("someting bad"); // good

    // ...

    throw std::invalid_argument("i is not even"); // good
    // enum classes are also allowed:
    enum class alert {RED, YELLOW, GREEN};
    throw alert::RED; // good

**Note**

Using user-defined exceptions, you can clearly classify exception types, so as
to understand the actual meaning and generation location of exceptions. Because
the definition of exceptions in the standard library is too broad, it is not
recommended to throw the exception types in the standard library directly.
However, you can define your own exceptions by inheriting these standard
exception types.

Throw by value, catch exceptions from a hierarchy by reference
**************************************************************

**Reason**

Throwing by value (not by pointer) and catching by reference prevents copying,
especially slicing base sub objects. For the object slicing, please refer to:
`What is object slicing`_

.. _What is object slicing: <https://stackoverflow.com/questions/274626/what-is-object-slicing#:~:text=THE%20DEFINITION%20OF%20SLICING%20PROBLEM,or%20parameter)%20of%20type%20Base.&text=Although%20the%20above%20assignment%20is,is%20called%20the%20slicing%20problem.>`_

**Bad example**

.. code-block:: c++

    void f()
    {
        try {
            // ...
            throw new widget{}; // don't: throw by value not by raw pointer, otherwise,
            // a memory leak will occur
            // ...
        }
        catch (base_class e) {  // don't: might slice
            // ...
        }
    }

Instead, use a reference:

**Good example**

.. code-block:: c++

    catch (base_class& e) { /* ... */ }
    // or - typically better still - a const reference:

    catch (const base_class& e) { /* ... */ }
    // Most handlers do not modify their exception and in general we recommend use of const.

**Note** Catch by value can be appropriate for a small value type such as an
enum value. To re-throw a caught exception use throw, not throw e. Using throw e
would throw a new copy of e (sliced to the static type std::exception) instead
of re-throwing the original exception of type std::runtime_error.

Destructors, deallocation, and swap must never fail
***************************************************

**Reason**

We don't know how to write reliable programs if a destructor, a swap, or a
memory de-allocation fails; that is, if it exits by an exception or simply
doesn't perform its required action.

**Bad example**

.. code-block:: c++

    class Connection {
        // ...
    public:
        ~Connection()   // Don't: very bad destructor
        {
            if (cannot_disconnect()) throw I_give_up{information};
            // ...
        }
    };

**Discussion**

For that, here are the discussions about the this topic: `discussion`_

.. _discussion: <http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#discussion-destructors-deallocation-and-swap-must-never-fail>`_

Don't try to catch every exception in every function
****************************************************

**Reason**

Catching an exception in a function that can not take a meaningful recovery
action leads to complexity and waste. Let an exception propagate until it
reaches a function that can handle it. Let cleanup actions on the unwinding path
be handled by RAII.

**Bad example**

.. code-block:: c++

    void f()   // bad
    {
        try {
            // ...
        }
        catch (...) {
            // no action
            throw;   // propagate exception
        }
    }

**Note**

Only catch the exceptions that can be handled. For those that cannot be handled,
such as no file access permission and memory application failure, these
exceptions do not need to be caught. An exception that is not caught will cause
the program to actively call std::terminate() to abort, it would be helpful to
find the issue as soon as possible.

Properly order your catch-clauses
*********************************

**Reason**

When the capture of base class precedes the capture of derived class, the
exception type of derived class cannot be captured. Therefore, the capture of
sub-classes should be put on the top.

**Good example**

.. code-block:: c++

    void f()
    {
        // ...
        try {
                // ...
        }
        catch (Derived& d) { /* ... */ }// Here, d can be caught if the exception type is Derived
        catch (Base& d) { /* ... */ }
        catch (std::exception& e) { /* ... */ }
        catch (...) { /* ... */ }
    }


**Bad example**

.. code-block:: c++

    void f()
    {
        // ...
        try {
                // ...
        }
        catch (Base& b) { /* ... */ }
        catch (Derived& d) { /* ... */ } //Here, d can't never be caught if the exception type is Derived
        catch (...) { /* ... */ }
        catch (std::exception& e) { /* ... */ }
    }


Don't use exception specifications
**********************************

**Reason**

Exception specifications make error handling brittle, impose a run-time cost,
and have been removed from the C++ standard.

**Bad example**

.. code-block:: c++

    int use(int arg) throw(X, Y)//bad, not recommend since C++11
    {
        // ...
        auto x = f(arg);
        // ...
    }

**Good example**

.. code-block:: c++

    int use(int arg) throw()//good, equal to noexcept
    {
        // ...
        auto x = f(arg);
        // ...
    }

**Note**

Exception specification specifies the types of exceptions that can be thrown by
a function. It has been deprecated since c++11. It will be warned during
compilation and removed from c++17. Currently, only throw() is supported and is
equivalent to noexcept.

Don't define a joinable std::thread in a try block
**************************************************

**Reason**

Define a joinable std::thread in try block, which may cause the exception never be caught.

**Bad example**

.. code-block:: c++

    class A
    {
    public:
        A(){ throw runtime_error("construct A failed.");}
        ~A(){}
    };

    int main()
    {
        try
        {
            std::thread t1(f1);   // f1 is a thread handler
            A a;        // create an object 'a', which will throw a runtime_error
            t1.join();  // the thread join could not be executed
        }
        catch(const runtime_error& e)
        {   // the runtime_error will never be caught
            std::cout << e.what() << "\n";
        }
        return 0;
    }

When execute the code "A a", a runtime_error will throw, and the thread object
't1' will deconstruct before exit the try block. but if the thread's state is
'joinable' when deconstructing, the 'std::terminate' will be called immediately,
which cause the following catch could not be executed.

Part of source code 'std::thread'

.. code-block:: c++

    class thread
    {
    public:
      thread() noexcept = default;

      template<typename _Callable,
                typename... _Args,
                typename = _Require<__not_same<_Callable>>>
      explicit thread(_Callable&& __f, _Args&&... __args) {
            //...
      }

      ~thread() {
        if (joinable())
          std::terminate();
      }

      //...
    };

**Good example**

.. code-block:: c++

    int main()
    {
        try
        {
            A a;        // create object a, which will throw a runtime_error
        }
        catch(const runtime_error& e)
        {
            std::cout << e.what() << "\n";
        }
        std::thread t1(f1);   // f1 is a thread handler
        t1.join();
        return 0;
    }

Exceptions versus assertions
****************************

Exceptions and asserts are two distinct mechanisms for detecting run-time errors
in a program. Use assert statements to test for conditions during development
that should never be true if all your code is correct. There's no point in
handling such an error by using an exception, because the error indicates that
something in the code has to be fixed. It doesn't represent a condition that the
program has to recover from at run time. An assert stops execution at the
statement so that you can inspect the program state in the debugger.

An exception continues execution from the first appropriate catch handler. Use
exceptions to check error conditions that might occur at run time even if your
code is correct, for example, "file not found" or "out of memory." 

Exceptions can handle these conditions, even if the recovery just outputs a
message to a log and ends the program. Always check arguments to public
functions by using exceptions. Even if your function is error-free, you might
not have complete control over arguments that a user might pass to it.

Exceptions and performance
**************************

The exception mechanism has a minimal performance cost if no exception is
thrown. If an exception is thrown, the cost of the stack traversal and unwinding
is roughly comparable to the cost of a function call. Additional data structures
are required to track the call stack after a try block is entered, and
additional instructions are required to unwind the stack if an exception is
thrown. However, in most scenarios, the cost in performance and memory footprint
isn't significant. The adverse effect of exceptions on performance is likely to
be significant only on memory-constrained systems. Or, in performance-critical
loops, where an error is likely to occur regularly and there's tight coupling
between the code to handle it and the code that reports it. In any case, it's
impossible to know the actual cost of exceptions without profiling and
measuring. Even in those rare cases when the cost is significant, you can weigh
it against the increased correctness, easier maintainability, and other
advantages that are provided by a well-designed exception policy.

Reference material
##################

- http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-errors
- https://www.geeksforgeeks.org/what-is-class-invariant/
- https://learn.microsoft.com/en-us/cpp/cpp/errors-and-exception-handling-modern-cpp?view=msvc-170

