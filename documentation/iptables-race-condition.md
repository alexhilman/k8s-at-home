# `iptables` Race Condition (1.6.1 and earlier)

There is an old bug in `iptables` which causes NAT translation to fail. A very detailed picture of the issue has been laid out by [Maxime Lagresle on tech.xming.com](https://tech.xing.com/a-reason-for-unexplained-connection-timeouts-on-kubernetes-docker-abd041cf7e02)

## Summary

When any request goes from one subnet to another over `iptables` the module is required to do address translation. To do this, `iptables` checks for an available ephemeral port for the network addresses it controls.

For example, to handle an outbound request from a `docker` container on `172.16.0.2/24` to an external address of `192.168.0.1/24` a NAT traversal takes place:
1. `iptables` will open an ephemeral port on the interface on its interface of `192.168.0.10/24`, lets say this is port `:33000`
1. `iptables` then translates the traffic from the `172.16.0.2/24` so it appears as if the traffic is coming from the source of `192.168.0.10:33000`
1. when a response comes back from the service, `iptables` handles the response translation as well

This ephemeral port reservation is where the bug lies: there is a non-atomic get-and-set operation for an available ephemeral port. To `iptables`, the process is as follows:
1. Check internal reservation tables for the next available port (say `33000`)
1. If available, reserve `33000` for the client

Unfortunately there is no lock around those two steps, so in a multi-threaded environment (read: any modern system) two requests could come in at the same time. This results in one of the requests reserving the port and the other getting denied.
