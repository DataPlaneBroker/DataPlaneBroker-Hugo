---
title: "Configuration & invocation"
date: 2020-01-15T14:38:33Z
draft: false
weight: 3
summary: "How to configure and invoke DPB components"
---

Here are details of how to configure and invoke DPB components.

## Corsa controller configuration

The Ryu controller `portslicer.py` is intended for use with Corsa DP2000-series switches.  The advantages of the Corsa fabric include the following:

* The OpenFlow rules are relatively simple, as Corsa's tunnel attachments handle VLAN tagging and detagging.  As far as the OF switch is concerned, every circuit comes in on its own ofport.

* The tunnel attachments also handle bandwidth shaping (on exit from the switch) and metering (on entry).  These are set up by the server through the Corsa's REST-based management interface, not by `portslicer.py`.

One instance of this controller can operate several Corsa VFCs independently, which is useful if you're emulating a wide-area network in a single physical switch.  If, however, you have several widely distribted switches, we recomment that you run a separate, co-located controller for each.

The `portslicer.py` controller presents two interfaces:

* An OpenFlow interface, which bridges within the Corsa will contact when set up

* A northbound interface (NBI), through which the broker expresses intents to the switch

Configuration amounts to the specification of the addresses of these interfaces, and are placed in the command line when invoking.  The examples here use the following addresses for the interfaces:

* `172.31.31.1:6556` for the OpenFlow interface — The netns that switches run in must be able to reach this address.  We'll use `dpb-netns` as an example.

* `localhost:8081` for the NBI — The DPB server must be able to reach this, and `localhost` is good enough if the controller and the broker are running in the same machine.

These details also go in the `ctrl` section of a fabric agent within the server configuration.

### Invoking the Corsa controller with `docker`

If you're using a Docker image, you can run it as follows, adjusting the port exposure as required:

```
docker run -d --restart unless-stopped --name dpb-ctrl-corsa \
  -p 127.0.0.1:8081:8081 -p 172.31.31.1:6556:6556 \
  simpsonst/dpb-ctrl-corsa
```

If you want to use different ports, **the second port in each pair must not change** — that's the internal port that the container binds to.


### Invoking the Corsa controller manually

To run the controller manually:

```
/usr/bin/ryu-manager \
	--ofp-listen-host 172.31.31.1 \
	--ofp-tcp-listen-port 6556 \
	--wsapi-host localhost \
	--wsapi-port 8081 \
	/usr/local/share/dataplane-broker/portslicer.py
```

Some (older?) versions of `ryu-manager` will just quit if the ports are already in use, making the command safe to run as a cronjob; bizarrely, newer versions seem to just hang about for ever, making it useless for `cron`.  In the latter case, perhaps an `@reboot` cronjob is appropriate.



## Generic OpenFlow controller configuration

The Ryu controller `tupleslicer.py` is intended for use with OpenFlow1.3 switches.

One instance of this controller can operate several switches independently, which is useful if you're emulating a wide-area network over several physical switches in a laboratory.  If, however, you have several widely distribted switches, you should run a separate, co-located controller for each.

The `tupleslicer.py` controller presents two interfaces:

* An OpenFlow interface, which the switch must be configured to contact

* A northbound interface (NBI), through which the broker expresses intents to the switch

Configuration amounts to the specification of the addresses of these interfaces, and are placed in the command line when invoking.  The examples here use the following addresses for the interfaces:

* `172.31.31.1:6556` for the OpenFlow interface — The netns that switches run in must be able to reach this address.  We'll use `dpb-netns` as an example.

* `localhost:8081` for the NBI — The DPB server must be able to reach this, and `localhost` is good enough if the controller and the broker are running in the same machine.

These details also go in the `ctrl` section of a fabric agent within the server configuration.

### Invoking the Generic OpenFlow controller manually

To run the controller manually:

```
/usr/bin/ryu-manager --ofp-listen-host 172.31.31.1 \
	--ofp-tcp-listen-port 6556 \
	--wsapi-host localhost --wsapi-port 8081 \
	/usr/local/share/dataplane-broker/tupleslicer.py
```

Some (older?) versions of `ryu-manager` will just quit if the ports are already in use, making the command safe to run as a cronjob; bizarrely, newer versions seem to just hang about for ever, making it useless for `cron`.  In the latter case, perhaps an `@reboot` cronjob is appropriate.


## DPB server configuration

Create server-side configuration as required in `~/.config/dataplanebroker/server.properties` (a Java 'properties' file, consisting of `dotted.name=value pairs`).  This file must contain a property called `agents` whose value is a comma-separated list of prefixes of other properties.  For example, if it contains `…, foo, …`, then there should also be a set of properties beginning `foo.`.

Here's an example server-side configuration, establishing three persistent switches (referred to as `london`, `athens` and `paris`, all realised on a single Corsa) and one persistent aggregator `aggr`:

```
## Two agents are required for each switch, one to implement
## a generic switch abstraction, and one to adapt it to the
## physical fabric.
agents=london-switch, london-fabric, \
paris-switch, paris-fabric, \
athens-switch, athens-fabric, \
aggr

aggr.name=aggr
aggr.type=persistent-aggregator
aggr.db.service=jdbc:sqlite:/home/me/.local/var/dataplane-broker/aggr.sqlite3

## The london switch stores its service status in an Sqlite3 database,
## and operates the fabric provided by the london-fabric agent.
london-switch.name=london
london-switch.type=persistent-switch
london-switch.db.service=jdbc:sqlite:/home/me/.local/var/dataplane-broker/london-switch.sqlite3
london-switch.fabric.agent=london-fabric

## The london fabric operates a single VFC in a Corsa,
## multiplexing services across it.  It is told how to
## operate the Corsa's REST management interface, what
## OpenFlow configuration to apply to the VFC (the
## controller address, and netns, if it needs to create
## it), and how to configure the VFC's controller (through
## a custom REST API).

london-fabric.name=london-fabric
london-fabric.type=corsa-dp2x00-sharedbr
london-fabric.description.prefix=dpb:vc:london:
london-fabric.subtype=openflow
london-fabric.rest.inherit=\#mycorsa.rest
london-fabric.ctrl.inherit=\#mycorsa.ctrl

## Other switches and fabrics follow a similar pattern.
athens-switch.name=athens
athens-switch.type=persistent-switch
athens-switch.db.service=jdbc:sqlite:/home/me/.local/var/dataplane-broker/athens-switch.sqlite3
athens-switch.fabric.agent=athens-fabric

athens-fabric.name=athens-fabric
athens-fabric.type=corsa-dp2x00-sharedbr
athens-fabric.description.prefix=dpb:vc:athens:
athens-fabric.subtype=openflow
athens-fabric.rest.inherit=\#mycorsa.rest
athens-fabric.ctrl.inherit=\#mycorsa.ctrl

paris-switch.name=paris
paris-switch.type=persistent-switch
paris-switch.db.service=jdbc:sqlite:/home/me/.local/var/dataplane-broker/paris-switch.sqlite3
paris-switch.fabric.agent=paris-fabric

paris-fabric.name=paris-fabric
paris-fabric.type=corsa-dp2x00-sharedbr
paris-fabric.description.prefix=dpb:vc:paris:
paris-fabric.subtype=openflow
paris-fabric.rest.inherit=\#mycorsa.rest
paris-fabric.ctrl.inherit=\#mycorsa.ctrl

## These credentials are used to operate a Corsa, and can
## be shared by multiple simulated switch abstractions.
mycorsa.rest.location=https:\//10.11.12.13/
mycorsa.rest.cert.file=mycorsa-certificate.pem
mycorsa.rest.authz.file=mycorsa-authz
mycorsa.ctrl.host=172.31.31.1
mycorsa.ctrl.port=6556
mycorsa.ctrl.netns=dpb-netns
mycorsa.ctrl.rest.location=http:\//localhost:8081/slicer/
```

The switch networks each operate a distinct bridge in the same Corsa.  The bridges all connect to the same controller on the same host.  As the networks access the same Corsa, and the same OpenFlow controller, their control parameters are all inherited from the same configuration, `mycorsa`.  `mycorsa.ctrl` identifies the northbound interface of the OF controller; `mycorsa.rest` identifies the management API of the Corsa.  You will need to enable this on your Corsa, as well as generate credentials to reference from the `rest.cert.file` and `rest.authz.file` fields.  (These filenames are relative to the file referencing them.)

Each switch network type is `corsa-dp2x00-sharedbr`, meaning a single bridge is created and used for all services.  Each network gives its bridge a distinct description within Corsa management; for example, the `paris` bridge's description begins with `dpb:vc:paris:`.  When the network first contacts the Corsa, it ensures that only one bridge has a description with that prefix.

In the `corsa-dp2x00-sharedbr` implementation, each active service of *N* end points manifests as *N* tunnel attachments to *N* available ofports of the bridge (which can be viewed on the Corsa with `show bridge br2 tunnel`, assuming `br2` is the bridge operated by the logical switch on whose behalf the controller acts).  When *N*=2, two OpenFlow rules can be seen to exchange traffic from each of these ofports to the other (with `show openflow flow br2`).  For larger *N*, an OpenFlow group is created to output to all ofports forming the service (`show openflow group br2), learning-switch rules based on this group are added, and unrecognized source MACs are delivered to the controller.

There is no topology information in the configuration file, as that is set up through other [dpb-client commands](http://scc-forge.lancaster.ac.uk/javadoc/dataplanebroker-test/uk/ac/london/networks/apps/Commander-method-main/1java$lang$String).  The configuration file only contains management connectivity information.


### SSH authorization

The server is managed through SSH.  The script `dpb-ssh-agent` is invoked on the server side, with minimal argumentation, then the client and server engage in a bidirectional exchange of JSON messages.  To allow a user access, take their SSH public key, and append it as a single line to `~/.ssh/authorized_keys` on the server account.  This should be augmented with settings that restrict what the client can invoke:

```
command="/usr/local/bin/dpb-ssh-agent +N -s /tmp/dataplane-broker.sock -m london -m athens -m paris -m aggr",no-port-forwarding,no-agent-forwarding,no-X11-forwarding,no-pty ecdsa-sha2-nistp256 AAAAE2VjZH…f85P+s= foo@example
```

(You might not need `+N` if you've got a newer `nc` command.)

The example above permits the caller to manage and control all four network abstractions (`london`, `athens`, `paris` and `aggr`).  Management operations include the addition and removal of terminals, and the additional, removal and configuration of trunks (for aggregators only).  Control operations include the creation, definition, activation, deactivation and release of services.  Use `-c` instead of `-m` to permit only control of a particular network.

If you're going to run the server in a Docker image, invoke the agent within the running container:

```
command="docker exec -i dpb-server /usr/local/bin/dpb-ssh-agent -o \"$SSH_ORIGINAL_COMMAND\" -N -s /tmp/dataplane-broker.sock -m root",no-port-forwarding,no-agent-forwarding,no-X11-forwarding,no-pty ecdsa-sha2-nistp256 AAAAE2VjZH…f85P+s= foo@example
```



## DPB server invocation

Run the DPB server process through Usmux:

```
usmux -d -B /tmp/dataplane-broker.sock -- dpb-server -s '%s' -L/usr/share/java -L/usr/local/share/java -lsqlite-jdbc -lhttpcore -lhttpclient -lcommons-logging
```

Everything after `--` is the actual DPB server command.  Usmux creates a Unix-domain socket `/tmp/dataplane-broker.sock` (which is also an argument of the SSH configuration below), listens for connections on it, and relays them to the given process.

The `-d` runs in debug mode, i.e., the server process remains in the foreground, and remains connected to the terminal.  Without it, the command exits as soon as the server is established and has forked into the background.

To run as a cronjob, remember to escape the `%` character in the command, as this is special in `crontab`:

```
*/10 * * * * usmux -B /tmp/dataplane-broker.sock -- dpb-server -s '\%s' -L/usr/share/java -L/usr/local/share/java -lsqlite-jdbc -lhttpcore -lhttpclient -lcommons-logging > /dev/null 2>&amp;1
```


If you're using the Docker image, run with:

```
docker run -d --restart unless-stopped --name dpb-server \
  -p 4753:4753 \
  -v ~/.config/dataplane-broker:/root/.config/dataplane-broker:ro \
  -v ~/.local/var/dataplane-broker:/root/.local/var/dataplane-broker:rw \
  simpsonst/dpb-server
```



## DPB client configuration

If you haven't got one, you'll need an SSH key to get in to remote servers:

```
ssh-keygen -t ecdsa -f ~/.ssh/dpb_ecdsa
```

That creates `dpb_ecdsa` and `dpb_ecdsa.pub` in your SSH directory.  Provide the public key to the server admin to install as noted above.

(You could use your existing key pair, but if you already use it to get unrestricted access to the account, using that key pair will preclude it, as the `authorized_keys` configuration will restrict the user to running only `dpb-ssh-agent`, and only in one particular way.)

Store client-side configuration in `~/.config/dataplanebroker/client.properties`.  The syntax is the same as for the server side.  However, all the agents are just SSH clients into the server account (`me@dpb-server.home` in this example).  For example:

```
## Define how to SSH to the server host and account.
beta.host=dpb-server.home
beta.user=me
beta.key-file=${user.home}/.ssh/london-switch_ecdsa
beta.config-file=/dev/null

## Indicate the configuration prefixes of agents to instantiate.
agents=london, paris, athens, aggr

## Define those agents with corresponding prefixes (e.g., "paris.")...

## This agent provides access to a remote switch via SSH.
paris.type=ssh-switch
paris.ssh.inherit=\#beta

## Identify this switch by this name.  This applies to both the
## command line (i.e., "-n paris"), and the name used at the remote
## site.
paris.name=paris

## Other networks are configured similarly...

athens.type=ssh-switch
athens.ssh.inherit=\#beta
athens.name=athens

london.type=ssh-switch
london.ssh.inherit=\#beta
london.name=london

aggr.type=ssh-switch
aggr.ssh.inherit=\#beta
aggr.name=aggr
```

Run the client as `dpb-client`.

If you've got the Docker image, you should create an alias:

```
function dpb-client () {
    docker run --rm --init \
      -v ~/.ssh:/root/.ssh:ro \
      -v ~/.config/dataplane-broker:/root/.config/dataplane-broker:ro \
      simpsonst/dpb-client "$@" 
}
```

You will likely need the `config-file` of the SSH options if running through Docker, as SSH will complain about ownership of the default configuration file.  (It's running as `root`, but the file is owned by `me`.)  Of course, this means you can't use any settings from that file.
