========================================
Sample Test Example in RST documentation
========================================

.. raw:: html

   <!-- This class allows us to hide the test setup part -->
   <style type="text/css">.hidden { display: none; }</style>

.. contents::
    :local:

This example demonstrates how RobotFramework tests can be embedded in
ReStructuredText documentation.  If you are converting an existing
tab-separated test suite, convert tabs to 4-spaces within the RST file.

The testcases should be in `code:: robotframework` blocks.

Installing RobotFramework
=========================

To execute this test, setup the following::

    pip install robotframework docutils Pygments
    git clone https://github.com/arista-eosplus/robotframework-aristalibrary.git
    cd robotframework-aristalibrary/
    python setup.py install


Executing tests
===============

Start tests using one of the examples, below::

    robot demo/sample_grammar.rst

    robot --variable SW1_HOST:localhost --variable SW1_PORT:61080 \
          --variable USERNAME:eapiuser --variable PASSWORD:icanttellyou \
          demo/sample_grammar.rst

    robot --variablefile demo/myvariables.py --noncritical not_enabled
          demo/sample_grammar.rst

Variable files
--------------

Variable files are just python modules with KEY = value pairs.

Example `myvariables.py`::

    """ My custom values for this test suite
    """

    SW1_HOST = 'localhost'
    SW1_PORT = 61080
    USERNAME = 'eapiuser'
    PASSWORD = 'icanttellyou'

    MLAG_DOMAIN = 'my_lag_domain'

Suite Setup
===========

.. code:: robotframework
   :class: hidden

    *** Settings ***
    Documentation    This is a sample Robot Framework suite which takes advantage of
    ...    the AristaLibrary for communicating with and controlling Arista switches.
    ...    Run with:
    ...    pybot --pythonpath=AristaLibrary --noncritical not_enabled demo/sample_grammar.rst

    Library    AristaLibrary.py
    Library    Collections
    Library    String
    Suite Setup    Connect To Switches
    Suite Teardown    Clear All Connections
    #Default Tags    mlag

    *** Variables ***
    ${TRANSPORT}    http
    ${SW1_HOST}    localhost
    ${SW1_PORT}    2080
    ${USERNAME}    vagrant
    ${PASSWORD}    vagrant

    *** Keywords ***
    Connect To Switches
        [Documentation]    Establish connection to a switch which gets used by test cases.
        Connect To    host=${SW1_HOST}    transport=${TRANSPORT}    username=${USERNAME}    password=${PASSWORD}    port=${SW1_PORT}
        #Run Keyword If   "${result['state']}" == 'disabled'  Fail   MLAG not enabled on this device  not_enabled
        #Log    ${result}

    Domain should be ${expected}
        ${mlag}=    Mlag Domain
        Should Be Equal  ${mlag}  ${expected}

    Expect interface ${interface} enabled to be ${value}
        ${output}=    Enable    show interfaces ${interface}   json
        ${result}=    Get From Dictionary    ${output[0]}    result
        # Strip top 2 layers off of the JSON response
        #${keys}=    Get Dictionary Keys     ${result}
        #${key}=  Get From List   ${keys}  0
        #${things}=  Get From Dictionary     ${result}   ${key}
        #${keys}=    Get Dictionary Keys     ${things}
        #${key}=  Get From List   ${keys}  0
        #${things}=  Get From Dictionary     ${things}   ${key}
        #${interface}=   ${interface}.capitalize()
        # Now we're down to the meat
        Should Be Equal     ${result['interfaces']["${interface}"]['lineProtocolStatus']}   ${value}

    Expect ${show} Field ${field} To Equal ${value}
        ${show}=    Strip String   ${show}     characters="
        ${value}=    Strip String   ${value}     characters="
        Log     Show: ${show}
        ${output}=    Enable    ${show}   json
        ${result}=    Get From Dictionary    ${output[0]}    result
        # Strip top 2 layers off of the JSON response
        ${keys}=    Get Dictionary Keys     ${result}
        ${key}=  Get From List   ${keys}  0
        ${things}=  Get From Dictionary     ${result}   ${key}
        ${keys}=    Get Dictionary Keys     ${things}
        ${key}=  Get From List   ${keys}  0
        ${things}=  Get From Dictionary     ${things}   ${key}
        # Now we're down to the meat
        Should Be Equal     ${value}    ${things["${field}"]}

    Expect ${show} Field ${field} To Contain ${value}
        ${show}=    Strip String   ${show}     characters="
        ${value}=    Strip String   ${value}     characters="
        ${output}=    Enable    ${show}   json
        ${result}=    Get From Dictionary    ${output[0]}    result
        # Strip top 2 layers off of the JSON response
        ${keys}=    Get Dictionary Keys     ${result}
        ${key}=  Get From List   ${keys}  0
        ${things}=  Get From Dictionary     ${result}   ${key}
        ${keys}=    Get Dictionary Keys     ${things}
        ${key}=  Get From List   ${keys}  0
        ${things}=  Get From Dictionary     ${things}   ${key}
        # Now we're down to the meat
        Should Contain  ${things["${field}"]}   ${value}

    #Ping from Source ${src_int} To ${dst_ip} Passes
    #    ${output}=    Enable    ping ${dst_ip} source ${src_int}    text
    #    ${result}=    Get From Dictionary    ${output[0]}    result
    #    Log    ${result}
    #    ${match}    ${group1}=    Should Match Regexp    ${result['output']}    (\\d+)% packet loss
    #    Should Be Equal As Integers    ${group1}    0    msg="Packets lost percent is ${result}, not 0!!!"

    Ping from ${src_ip} To ${dst_ip} ${desired}
        ${output}=    Enable    ping ${dst_ip} source ${src_ip}    text
        ${result}=    Get From Dictionary    ${output[0]}    result
        Log    ${result}
        ${match}    ${group1}=    Should Match Regexp    ${result['output']}    , (\\d+)% packet loss
        Run Keyword If  "${desired}"=="Passes"  Should Be Equal As Integers    ${group1}    0    msg="Packets lost percent is ${result}, not 0!!!"
        Run Keyword If  "${desired}"=="Fails"  Should Be Equal As Integers    ${group1}    100    msg="Packets lost percent is ${result}, not 0!!!"

Test Cases
===============

.. code:: robotframework

    *** Test Cases ***
    Interface Ethernet1 status
        Expect interface Ethernet1 enabled To Be up

    Interface Ethernet1 description
        Expect "show interfaces ethernet1" Field description To Equal "peer-link member"

    Interface Ethernet1 description contains
        Expect "show interfaces ethernet1" Field description To Contain "member"

    Ping Test 1
        Ping From 10.0.2.15 To 10.0.2.2 Passes

    Ping Test 2
        Ping From Ma1 To 10.0.2.2 Passes

    Ping Test Fail
        Ping From Ma1 To 10.0.2.9 Fails
