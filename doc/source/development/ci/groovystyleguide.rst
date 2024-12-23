***********************************
Groovy Style Guide for CI
***********************************

.. contents:: :local:

1. Material
###########

`Official groovy style guide <https://groovy-lang.org/style-guide.html>`_ is a good read.

2. Use return keyword
#####################

Use return keyword even if it is optional to make it easier to see that
function returns something.

See `return keyword chapter in style guide <https://groovy-lang.org/style-guide.html#_return_keyword_optional>`_ for more information.

3. Use typed variables and functions
####################################

This makes code more readable, supports static type checking and makes
debugging easier. This is especially useful for public APIs.

So instead of:

.. code-block:: groovy

    def myFunction(mapArgument, stringArgument)

Use:

.. code-block:: groovy

    String myFunction(Map inputArgument, String stringArgument)

If function doesn't return anything use void:

.. code-block:: groovy

    void functionWhichDoesNotReturnAnything()

See `optional typing advice in style guide <https://groovy-lang.org/style-guide.html#_optional_typing_advice>`_ for more information.

4. Function naming
##################

Function name should tell what the function does.

Keeping functions short and following the
`single-responsibility principle <https://en.wikipedia.org/wiki/Single-responsibility_principle>`_
helps to achieve this.

5. Unit test formatting
#######################

For all new test cases follow the formatting from the
`spock primer examples <https://spockframework.org/spock/docs/2.1/spock_primer.html#_when_and_then_blocks>`_


No indentation after when/then/setup/given/.. and one empty line between them:

.. code-block::

    def "pushing an element on the stack"() {​​​​​​​​
        when:
        stack.push(elem)

        then:
        !stack.empty stack.size() == 1
        stack.peek() == elem
    }​​​​​​

Changing existing unit test cases to follow this formatting is optional,
as it would cause quite much whitespace changes, which may make it harder
to track changes.

6. Unit test naming
###################

Name of the unit test should tell what the test tries to verify. Some examples:

.. code-block::

    def "validateGitCommitEmail should pass when author email is valid" () {​​​
    def "validateGitCommitEmail should pass when there are no commits" () {​​​​​​​
    def "validateGitCommitEmail should fail when author email domain is invalid" () {​​​​​​​​​​​

It is also useful to start the test case name with the function name,
so that it is easy to run all tests for the functions with one command,
for example:

.. code-block::

    mvn verify -Dtest=GitUtilSpec#'validateGitCommitEmail*'
