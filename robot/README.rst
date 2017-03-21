=============================
Sample Output for Robot Tests
=============================

.. code:: shell

  $ robot --pythonpath=AristaLibrary --noncritical not_enabled  --variablefile demo/myvariables.py \
  sample_mlag_test.rst

  ==============================================================================
  Sample Mlag Test :: This is a sample Robot Framework suite which takes adva...
  ==============================================================================
  State via manual check                                                | PASS |
  ------------------------------------------------------------------------------
  Peer-link Is Up                                                       | PASS |
  ------------------------------------------------------------------------------
  Local Interface Is Up                                                 | PASS |
  ------------------------------------------------------------------------------
  Ports Are Not Error-disabled                                          | PASS |
  ------------------------------------------------------------------------------
  Ports are not disabled                                                | FAIL |
  One or more ports are disabled: 0 (integer) != 0 (string)
  ------------------------------------------------------------------------------
  All configured ports are active                                       | PASS |
  ------------------------------------------------------------------------------
  Peer address is reachable                                             | FAIL |
  'connect: Network is unreachable
  ' does not match '(\d+)% packet loss'
  ------------------------------------------------------------------------------
  Member ports are active                                               | FAIL |
  Resolving variable '${lags[[]]['members']}' failed: TypeError: list indices must be integers, not list
  ------------------------------------------------------------------------------
  State via method                                                      | PASS |
  ------------------------------------------------------------------------------
  Verify domain setting                                                 | PASS |
  ------------------------------------------------------------------------------
  Sample Mlag Test :: This is a sample Robot Framework suite which t... | FAIL |
  10 critical tests, 7 passed, 3 failed
  10 tests total, 7 passed, 3 failed
  ==============================================================================
  Output:  /Users/.../output.xml
  Log:     /Users/.../log.html
  Report:  /Users/.../report.html

