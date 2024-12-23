*************************************************************
Replacing robot cases with integration test cases within code
*************************************************************

.. contents:: :local:

Why and how to replace robot tests with integration tests
#########################################################

Not all integration test cases must be implemented as robot test cases, sometime 
integration test cases also can be implemented via code with way of black-box testing 
or white-box testing, such test cases will consume less resources, depend on less tools
and run faster than robot cases, also such test could be more stable and maintainable
than robot cases, if we explore more possibilities for replacing robot cases to 
integration test cases within code, then our PCI could be improve to run faster and 
stable.

Below case just tests the library of rcphttpc and rcphttpd, library rcphttpc provides the
interfaces to act as a http client, library rcphttpd provides the interfaces to act as a
http server. 

This case is quite simple to use a http client to connect to the http server, and the server
could be a sync http server or a async http server. We developed a tool rcphttptest to act
as a stub for http server and client. It is quite heavy to test such scenario via robot
framework, such case could be completely designed as integration test within code.

Robot example
#############

What's the case tested
**********************

Case link:
`_RCP1334_function_test.robot <https://gerrite1.ext.net.nokia.com/gitweb?p=scm_rcp/ELAMRobotTest.git;a=blob;f=robottests/ELAM/HTTPRestFramework/RCP1334_function_test.robot;h=18f5b4ad63ab95b733246f1cf03ba07115d15402;hb=refs/heads/rcp2.0>`_

Case name: 
RCP_RestAPI_003 connection without TLS and with IPV4

Case description:
The case is to check the connection between client and server using ipv4 and without TLS, 
it should be tagged as priority-must.

Execution description:

1. Start one async server, server ip is ipv4, and TLS is not enabled.
2. Start one async client, use method HEAD to send request, store the response to a file.
3. Start another sync server, server ip is ipv4, and TLS is not enabled.
4. Start another async client, use method HEAD to send request, store the response to a file.

Expected result:

1. start server successfully.
2. start client successfully. response status is OK, the response content is empty.
3. start server successfully.
4. start client successfully. the response status is error when requested resource does 
   not existed.


Robot case
**********
::

    [Setup]    RestAPI_003 setup
    Start A Sync Server Successfully    ${ip_v4}    ${ipv4_port}
    Start A Client Successfully ${server_ip}    ${server_port}    ${url}  ${content}
    Check Response From Client Side  ${status_200}
    Check Server Status
    Start A Async Server Successfully   ${ip_v4}    ${second_ipv4_port}
    Start A Client Successfully   ${server_ip}    ${server_port}    ${url}  ${content}
    Check Response From Client Side  ${status_200}
    Check Server Status
    [Teardown]    common teardown    /tmp/case_003*


Integration test example with CppUtest
######################################

The stub server can be replaced with a stub class testAsyncServer which is implemented with 
async interfaces of rcphttpd, and client can be replaced with command tool curl. Above 
robot case could use two integration cases to cover, one test async server, one test sync
server. 

Below is the code that tests the async server with CppUtest.

.. code-block:: c++

    TEST(http_async_server_test, happypath_httpasyncserver_connection_test) 
    {
        //start a async server
        std::string ip = "127.0.0.1";
        std::string url = "/nokia/rcp/test";
        testAsyncServer* httpd = new testAsyncServer();
        bool excep = false;
        int listenPort = -1;
        try {
            httpd->addToCallbackMap();
            httpd->create(ip, 0);
            httpd->start();
            listenPort = httpd->listenPort;
        } catch (std::exception& ex) {
            excep = true;
        }
        CHECK_FALSE(excep);
        CHECK_TRUE(listenPort != 0);

        //Test connect server successfully
        std::string clientCmd = "curl -i -X GET http://" + ip +":" + std::to_string(listenPort) + "/nokia/rcp/test";
        std::cout<<"clientCmd == "<<clientCmd<<std::endl;
        std::string cmdResult = exec(clientCmd.c_str());
        std::cout<<"cmdResult:"<<cmdResult<<std::endl;
        auto pos = cmdResult.find("201");
        CHECK(pos != string::npos);
        pos = cmdResult.find("get method");
        CHECK(pos != string::npos);

        //Stop the server and release the resources
        httpd->stop();
        delete httpd;
    }

It's similar code to test sync http server.

Performance comparison
######################

The time for such test consume:
::

    time ./librcphttpasync_test_runner
    OK (6 tests, 6 ran, 15 checks, 0 ignored, 0 filtered out, 61 ms)
    real    0m0.084s
    user    0m0.030s
    sys     0m0.038s


The robot case in FCI needs about 6 seconds, so you can see that such integration test is much
quicker than robot case.
The actual reason for this robot case consuming much time is that there are lots of shell
commands executed remotely in the management node, each taking more than 100 ms, that
accounts for more than 4 seconds from the test execution time.
