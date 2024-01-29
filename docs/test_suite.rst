The Test Suite
==============

This page describes more details about the Test Suite class.

The test suite class is the main class for running tests.
Each test case is defined as a method in the test suite.
The method must start with :code:`test_`.
These test methods are executed by the test suite run method.

Preceding the test cases, an optional setup method is executed.
If the setup fails, execution is stopped.
Following the test cases a teardown method will be executed,
regardless whether the test cases passed or failed.

Test suite creation
-------------------

Creating a test suite is a simple as creating a subclass:

.. code-block:: python

    import lily_unit_test

    class MyTestSuite(lily_unit_test.TestSuite):
        ...

Test cases are added using methods with the prefix: :code:`test_`:

.. code-block:: python

    import lily_unit_test

    class MyTestSuite(lily_unit_test.TestSuite):

        def test_login(self):
            ...

        def test_upload_image(self):
            ...

In this case two test cases are defined.
The tests are executed in the order as they are created, from top to bottom.

Running the test suite
----------------------

The test suite can be run using the :code:`run` method.
The :code:`run` method returns :code:`True` if the test suite passed and :code:`False` if failed.
In order to make the test suite run properly, the test suite must be initialized:

.. code-block:: python

    # Initialize test suite, the test suite does not have any parameters
    ts = MyTestSuite()
    # Run the test suite
    ts.run()

    # A nice one liner
    MyTestSuite().run()

    # Using the test result
    if MyTestSuite().run():
        print("Yay, the test suite passed!")
    else:
        print("Oops, the test suite failed...")

Using setup and teardown
------------------------

The test suite has a default setup and teardown that can be overridden in the subclass.
The default setup and teardown do nothing, they are just empty methods.
If not overridden, it will not matter.
The setup and teardown can be overridden in your test suite:

.. code-block:: python

    import lily_unit_test

    class MyTestSuite(lily_unit_test.TestSuite):

        connection = None

        def setup(self):
            self.connection = connect_to_server(user, password)

        def test_upload_image(self):
            self.connection.upload_image(filename)

        def test_download_image(self):
            self.connection.download_image(uri, filename)

        def teardown(self):
            # In case the connection could not be created, the connection propery could still be None
            if self.connection is not None and self.connection.is_connected():
                self.connection.close()

In this hypothetical example, prior to all tests a connection to a server is created.
In case this fails because of an exception, the execution stops and the test suite fails.
In case the setup passes, the test cases will be executed.
Finally, the teardown is executed. The teardown closes the connection with the server.
If in the hypothetical case, the connection was not established in the setup (failed for some reason),
closing a not established connection can cause an exception.
The test suite will fail if the teardown fails because of an exception.

Making test suites pass or fail
-------------------------------

A test case method or setup method is passed by the following conditions:

* There were no exceptions or asserts.
* The return value is None or True.

A test case method or setup method is failed by the following conditions:

* An exception or assert was raised
* The return value is False

The teardown can only fail if an exception or assert was raised. The return value is not used.

Examples of passing or failing test suites
------------------------------------------

The following examples only show the specific test method from the test suite.

.. code-block:: python

    def test_login(self):
        # Setup that fails by exception from the connect to server method
        self.connection = connect_to_server(user, password)
        # The return value is by default None

    def test_login(self):
        self.connection = connect_to_server(user, password)
        # Fail by raising an exception
        if not self.connection.is_connected():
            raise Exception("We are not connected")
        # The return value is by default None

    def test_login(self):
        self.connection = connect_to_server(user, password)
        # Fail by assert
        assert self.connection.is_connected(), "We are not connected"
        # The return value is by default None

    def test_login(self):
        self.connection = connect_to_server(user, password)
        # Pass or fail by return True or False
        return self.connection.is_connected()

Logging messages
----------------

The test suite has a build in logger for logging messages.
Log messages are stored in an internal buffer (a list with strings)
and are directly written to the standard output (stdout, usually the console).
Messages from the standard output and error handler (stdout and stderr),
are redirected to the logger. When using :code:`print()`, the output is stored in the logger.
If an exception is raised, the trace message from the exception is stored in the logger.
The logger can be accessed by the log attribute of the test suite:

.. code-block:: python

    import lily_unit_test

    class MyTestSuite(lily_unit_test.TestSuite):
        ...
        some test stuff here
        ...


    # Initialize the test suite
    ts = MyTestSuite()

    # Access the logger
    # Write a message to the internal buffer and it will also shown on the standard output
    ts.log.info("Log an info level message")

    print("This mesage will be written to the logger")

    # Get a reference to the list with messages
    messages = ts.log.get_log_messages()

    # Get a copy of the list with messages
    messages = ts.log.get_log_messages().copy()

Note that you do not need to run the test suite to use the logger.

Below some examples of log messages.

.. code-block:: python

    def test_login(self):
        # Info message
        self.log.info("Connect to server")
        self.connection = connect_to_server(user, password)

        # Debug message
        self.log.debug("Connection status: {}".format(self.connection.is_connected())

        # Let's check the connection properties using print
        # These messages will be written automatically to the logger
        # This can be useful for a quick logging of some variables
        print("Server IP  :", self.connection.get_server_ip())
        print("Server name:", self.connection.get_server_name())

        # Insert an empty line
        self.log.empty_line()

        if not self.connection.is_connected()
            # Error message
            self.log.error("We are not connected")

        return self.connection.is_connected()

Note that logging an error message NOT automatically makes the test fail.

See the logger API documentation for more details.

Classification
--------------

The test suite object has a build in classification.
This can be set by the :code:`CLASSIFICATION` attribute.

.. code-block:: python

    import lily_unit_test

    class MyTestSuite(lily_unit_test.TestSuite):

        CLASSIFICATION = <value>

The values are defined in an object called :code:`Classification` and can be imported from the package.

.. code-block:: python

    import lily_unit_test

    # Regular test suite
    class MyTestSuite01(lily_unit_test.TestSuite):

        # By default the value is PASS, so this is not necessary
        CLASSIFICATION = lily_unit_test.Classification.PASS


    # Test suite that we expect to fail
    class MyTestSuite02(lily_unit_test.TestSuite):

        # Override the default value
        CLASSIFICATION = lily_unit_test.Classification.FAIL

The default value is :code:`PASS`, and is usually suitable for most test suites.
This means in general there is no need to override this attribute.
Setting this attribute to :code:`FAIL` will make the test suite pass in case of a failure.
All errors are logged as usual but the end result will be passed in case of a failure.
If the test suite passes, the test suite is marked as failed.

This situation is useful when the test fails because of a known issue,
and you want to accept the known issue. As long as the issue is there the test will pass.
When the issue is solved, the test fails, reminding you to restore the classification attribute.

The log messages will show this:

.. code-block:: console

    - No classification defined:
    2024-01-05 19:35:54.328 | ERROR  | Test classification is not defined: None
    2024-01-05 19:35:54.328 | ERROR  | Test suite TestSuiteClassification: FAILED

    - Classification set to FAIL and test suite fails because of a known issue, but is accepted
    2024-01-05 19:38:17.989 | INFO   | Test suite failed, but accepted because classification is set to 'FAIL'
    2024-01-05 19:38:17.989 | INFO   | Test suite TestSuiteClassification: PASSED

    - Classification set to FAIL and test suite passes because of the known issue is solved
    2024-01-05 19:39:46.530 | ERROR  | Test suite passed, but a failure was expected because classification is set to 'FAIL'
    2024-01-05 19:39:46.530 | ERROR  | Test suite TestSuiteClassification: FAILED

Test suite methods
------------------

The test suite has some build-in methods that can be used in the test methods.

.. currentmodule:: lily_unit_test

.. automethod:: TestSuite.fail
    :no-index:

The fail methods logs an error message and raises an exception. By default, an exception is raised.
When the exception is raised, the test suite stops and is reported as failed.
Setting the :code:`raise_exception` to False, does not raise an exception and the test suite continues.
The fail method always returns :code:`False`.

.. code-block:: python

    class MyTestSuite(lily_unit_test.TestSuite):

        def test_something(self):
            ...
            do some things
            ...

            # In case something is wrong and we cannot continue.
            if not check_something_that_must_be_good():
                # Log a failure with exception, this will make the test suite fail.
                self.fail("Something is wrong and we cannot continue")

            # In case something is wrong and we still can continue.
            result = passed
            if not check_if_something_is_ok():
                # Log a failure without exception, this will not make the test suite fail.
                # To make it fail, we use the result value and return it later when we are done.
                result = self.fail("Something is not OK, but we continue", False)

            ...
            do some other stuff
            ...

            # Return whether we passed or failed
            return result

.. automethod:: TestSuite.fail_if
    :no-index:

If the expression evaluates to :code:`True`, an error message is logged and exception is raised by default.
When the exception is raised, the test suite stops and is reported as failed.
Setting the :code:`raise_exception` to False, does not raise an exception and the test suite continues.
The fail_if method returns :code:`False` if the expression is :code:`True`.

Let's rewrite the test method in the previous example, but now using :code:`fail_if`.
The test looks more simpler now.

.. code-block:: python

    class MyTestSuite(lily_unit_test.TestSuite):

        def test_something(self):
            ...
            do some things
            ...

            self.fail_if(check_something_that_must_be_good(), "Something is wrong and we cannot continue")

            # Result will be False if the expressing is True, meaning a failure.
            result = self.fail_if(check_if_something_is_ok(), "Something is not OK, but we continue", False)

            ...
            do some other stuff
            ...

            # Return whether we passed or failed.
            return result