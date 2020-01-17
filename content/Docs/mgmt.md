---
title: "Management"
date: 2020-01-15T14:49:22Z
draft: false
weight: 4
summary: "Management operations include describing the network topology to the broker.  The SSH client is used to configure network topology."
---


Details of network topology are set up dynamically through the management client `dpb-client`.

# Example topology configuration

In our example, a single Corsa is used to host three bridges (a.k.a. VFCs).  Ports 24 and 25 are assumed to connect to (say) the dataplanes of two OpenStack installations.  Ports 13 and 29 are looped together at 10G to simulate a long-distance link.  All these ports must use tunnel mode `ctag`.  We wish to operate the bridges as a chain connecting the two OpenStack installations together.

The three bridges correspond to three logical switches (`athens`, `paris`, `london`) in the network abstraction.  `london` will connect directly to one of the OpenStacks, and `athens` will connect to the other.  Two trunks of 5G each will be multiplexed on the port 13-29 loop.  Each switch will have two terminals, named `openstack` if it connects to an OpenStack installation, or named the same as the switch it connects to otherwise.

| Logical network | Terminal | Mapping | Meaning |
| :- | :- | :- | :- |
| `london` | `openstack` | `phys.25` | Physical port 25 (OpenStack #1) |
| `london` | `paris` | `phys.13` | Physical port 13 (loop to 29) |
| `paris` | `london` | `phys.29` | Physical port 29 (loop to 13) |
| `paris` | `athens` | `phys.29` | Physical port 29 (loop to 13) |
| `athens` | `paris` | `phys.13` | Physical port 13 (loop to 29) |
| `athens` | `openstack` | `phys.24` | Physical port 24 (OpenStack #2) |
| `aggr` | `london-openstack` | `london:openstack` | Terminal `openstack` in network `london` |
| `aggr` | `athens-openstack` | `athens:openstack` | Terminal `openstack` in network `athens` |


The following command sets up all terminals and trunks in one go:

```
dpb-client -n london \
	   add-terminal openstack phys.25 \
	   add-terminal paris phys.13 \
	   -n paris \
	   add-terminal london phys.29 \
	   add-terminal athens phys.29 \
	   -n athens \
	   add-terminal paris phys.13 \
	   add-terminal openstack phys.24 \
	   -n aggr \
	   add-terminal athens-openstack athens:openstack \
	   add-terminal london-openstack london:openstack \
	   add-trunk athens:paris paris:athens \
	   add-trunk london:paris paris:london \
	   provide paris:athens 5000 \
	   provide paris:london 5000 \
	   open paris:athens 100-399 \
	   open paris:london 400-699
```

Each `-n` selects a logical switch to contact for subsequent commands, so first `london` is configured, then `paris`, and so on.  Terminals on the switches are created first.  This allows trunks in the aggregator to be created by referencing them.

The `provide` commands increase the bandwidth capacity on each trunk by 5000Mbps each.  Each trunk is set to use at most half of the physical capacity of the loop they share.

The `open` commands allow ranges of labels (which map to VLAN tags) to be used on each trunk.  By choosing disjoint ranges, they can share the same physical loop.
