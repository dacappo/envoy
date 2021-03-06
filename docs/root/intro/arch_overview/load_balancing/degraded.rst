.. _arch_overview_load_balancing_degraded:

Degraded endpoints
------------------

Envoy supports marking certain endpoints as degraded, meaning that they are able to receive
traffic, but should only receive traffic once there are not sufficient healthy hosts available.

Routing to degraded hosts can be thought of as routing to hosts in a lower 
:ref:`priority <arch_overview_load_balancing_priority_levels>`. As hosts in higher priorities go 
unhealthy, traffic spills down to lower priorities. As the amount of healthy hosts
available is no longer sufficient to handle 100% of the load, traffic will spill over to degraded 
hosts using the same mechanism as priority spillover for healthy hosts. This ensures that 
traffic is gradually shifted to degraded hosts as it becomes necessary.


+--------------------------------+------------------------------+-------------------------------+
| P=0 healthy/degraded/unhealthy | Traffic to P=0 healthy hosts | Traffic to P=0 degraded hosts |
+================================+==============================+===============================+
| 100%/0%/0%                     | 100%                         |   0%                          |
+--------------------------------+------------------------------+-------------------------------+
| 71%/0%/29%                     | 100%                         |   0%                          |
+--------------------------------+------------------------------+-------------------------------+
| 71%/29%/0%                     | 99%                          |   1%                          |
+--------------------------------+------------------------------+-------------------------------+
| 25%/65%/10%                    | 35%                          |   65%                         |
+--------------------------------+------------------------------+-------------------------------+
| 5%/0%/95%                      | 100%                         |   0%                          |
+--------------------------------+------------------------------+-------------------------------+

Endpoints can be marked as degraded by using active health checking and having the upstream host
return a :ref:`special header <arch_overview_health_checking_degraded>`.

Note: Degraded endpoints are considered unhealthy when computing panic thresholds. This means that
the number of unhealthy hosts plus the number of degraded hosts is above the panic threshold, the
containing priority will enter panic mode.
