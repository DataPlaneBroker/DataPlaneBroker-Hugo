---
title: "Control"
date: 2020-01-15T14:50:43Z
draft: false
weight: 5
---

The broker can be operated by SSH or REST.

# SSH interface

The command `dpb-client` invokes the broker over SSH.  `-n nw` selects the network `nw` for subsequent arguments to apply to.  `-s 4` selects service 4 for subsequent arguments.

## Operations on the network

* Test connectivity:

		dpb-client -n aggr dump

* Create a service:

		dpb-client -n aggr new

## Operations on existing services

* Select the service, and define its end points (100Mbps ingress):

		dpb-client -n aggr -s 1 -b 100 \
			-e athens-openstack:50 \
			-e london-openstack:80 initiate

	This will plot a spanning tree across the switches, and allocate bandwidth from the physical links.

* Activate the service, creating the forwarding rules in the switches:

		dpb-client -n aggr -s 1 activate

	In the case of a Corsa fabric, this also results in the attachment of VLAN tunnels to the bridge.

* De-activate the service:

		dpb-client -n aggr -s 1 deactivate

* Activate the service:

		dpb-client -n aggr -s 1 activate

* De-activate the service:

		dpb-client -n aggr -s 1 deactivate

* Activate the service:

		dpb-client -n aggr -s 1 activate

* De-activate the service:

		dpb-client -n aggr -s 1 deactivate

* Release the service (resulting in its destruction):

		dpb-client -n aggr -s 1 release

* Watch the service for events:

		dpb-client -n aggr -s 1 watch

[Full command-line details](http://scc-forge.lancaster.ac.uk/javadoc/dataplanebroker-test/uk/ac/lancs/networks/apps/Commander-method-main/1java$lang$String) are available.

## REST interface

[The REST interface is described in the Javadoc.](http://scc-forge.lancaster.ac.uk/javadoc/dataplanebroker-test/uk/ac/lancs/networks/rest/RESTNetworkControlServer)  Each network `foo` is served under a prefix `http://0.0.0.0:4753/network/foo`.
