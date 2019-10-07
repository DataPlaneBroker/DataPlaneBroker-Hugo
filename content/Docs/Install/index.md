---
title: 'Installation'
date: 2019-09-01T15:14:39+10:00
weight: 2
---

## Installing the DPB service

<p>The instructions assume a platform of Ubuntu 18.04.  It can surely be adapted for other Linux platforms.

<p>Install Java 9 or later:

<pre>
sudo apt-get install openjdk-11-jdk
</pre>

<p>Install other prerequisites:

<pre>
sudo apt-get install build-essential libjsonp-java junit4 libhttpcore-java \
libcommons-httpclient-java libcommons-logging-java gawk par subversion
</pre>

<p>To run Ryu controllers, you also need:

<pre>
sudo apt-get install python-ryu
</pre>

<p><strong>Ubuntu 16.04 doesn't have <samp>libjsonp-java</samp>.</strong>  This procedure should install it under <samp>/usr/local/share/java/</samp>:

<pre>
wget 'http://www.java2s.com/Code/JarDownload/javax.json/javax.json-1.0.jar.zip'
unzip javax.json-1.0.jar.zip
sudo mkdir -p /usr/local/share/java
sudo mv javax.json-1.0.jar /usr/local/share/java
sudo ln -s javax.json-1.0.jar /usr/local/share/java/javax.json.jar
</pre>

<p>Additionally for a DPB server installation, a database connector is required:

<pre>
sudo mkdir -p /usr/local/share/java
sudo curl 'https://bitbucket.org/xerial/sqlite-jdbc/downloads/sqlite-jdbc-3.23.1.jar' \
     -o /usr/local/share/java/sqlite-jdbc-3.23.1.jar
</pre>

<p>The DPB and its remaining prerequisites are best checked out from their repositories.  We'll assume you'll keep working copies in a directory such as <samp>~/works</samp>.

<p>Install Jardeps, which provides rules for compiling Java with makefiles:
<pre>
cd ~/works
svn co 'http://scc-forge.lancaster.ac.uk/svn-repos/misc/jardeps/trunk' jardeps
cd jardeps
make
sudo make install
</pre>

<p>Install Binodeps, which provides enhanced rules for build C and C++ programs using GNU Make:
<pre>
mkdir -p ~/works
cd ~/works
svn co http://scc-forge.lancaster.ac.uk/svn-repos/misc/binodeps/trunk/ binodeps
cd binodeps
make
sudo make install
</pre>



<p>
Install DDSLib, a C library for managing various data structures like linked lists, binary heaps, etc.:
<pre>
cd ~/works
svn co 'http://scc-forge.lancaster.ac.uk/svn-repos/utils/ddslib/branches/stable' ddslib
cd ddslib
cat > ddslib-env.mk &lt;&lt;EOF
CFLAGS += -O2 -g
CFLAGS += -std=gnu11
#CFLAGS += -fgnu89-inline

CPPFLAGS += -D_XOPEN_SOURCE=600
CPPFLAGS += -D_GNU_SOURCE=1
CPPFLAGS += -pedantic -Wall -W -Wno-unused-parameter
CPPFLAGS += -Wno-missing-field-initializers

CXXFLAGS += -O2 -g
CXXFLAGS += -std=gnu++11
EOF
make
sudo make install
</pre>

<p>
Install Reactor, a library for event-driven programming in C:
<pre>
cd ~/works
svn co 'http://scc-forge.lancaster.ac.uk/svn-repos/utils/react/trunk' react
cd react
cat > react-env.mk &lt;&lt;EOF
ENABLE_CXX=no
CFLAGS += -O2 -g
CFLAGS += -std=gnu11
#CFLAGS += -fgnu89-inline

CPPFLAGS += -D_XOPEN_SOURCE=600
CPPFLAGS += -D_GNU_SOURCE=1
CPPFLAGS += -pedantic -Wall -W -Wno-unused-parameter
CPPFLAGS += -Wno-missing-field-initializers

CXXFLAGS += -O2 -g
CXXFLAGS += -std=gnu++11
EOF
make
sudo make install
</pre>

<p>
Usmux allows a Java process to receive connections over a Unix-domain socket, and fork into the background.  The DPB server process makes itself available over such a socket.  Install Usmux:
<pre>
cd ~/works
svn co 'http://scc-forge.lancaster.ac.uk/svn-repos/misc/usmux/branches/export' usmux
cd usmux
cat > usmux-env.mk &lt;&lt;EOF
CFLAGS += -O2 -g
CFLAGS += -std=gnu11
CPPFLAGS += -D_XOPEN_SOURCE=600
CPPFLAGS += -D_GNU_SOURCE=1
CPPFLAGS += -pedantic -Wall -W -Wno-unused-parameter
CPPFLAGS += -Wno-missing-field-initializers
EOF
make
sudo make install
</pre>

<p>Install Java REST library (noting correct path for <samp>javax.json.jar</samp> if you installed it manually):
<pre>
cd ~/works
svn co 'http://scc-forge.lancaster.ac.uk/svn-repos/utils/lurest/trunk' \
  lurest
cat &gt; lurest-env.mk &gt;&gt;EOF
CLASSPATH += /usr/share/java/httpclient.jar
CLASSPATH += /usr/share/java/httpcore.jar
CLASSPATH += /usr/local/share/java/jardeps-lib.jar
<strong>CLASSPATH += /usr/share/java/javax.json.jar</strong>
EOF
make
sudo make install
</pre>

<p>Install DPB (noting correct path for <samp>javax.json.jar</samp> if you installed it manually):
<pre>
cd ~/works
git clone git@github.com:DataPlaneBroker/DPB.git
cd DPB
cat &gt; dataplanebroker-env.mk &lt;&lt;EOF
CLASSPATH += /usr/share/java/junit4.jar
<strong>CLASSPATH += /usr/share/java/javax.json.jar</strong>
CLASSPATH += /usr/share/java/httpcore.jar
CLASSPATH += /usr/share/java/httpclient.jar
CLASSPATH += /usr/share/java/commons-logging.jar
CLASSPATH += /usr/local/share/java/jardeps-lib.jar
CLASSPATH += /usr/local/share/java/usmux_session.jar
CLASSPATH += /usr/local/share/java/sqlite-jdbc-3.21.0.jar
CLASSPATH += /usr/local/share/java/lurest-server-api.jar
CLASSPATH += /usr/local/share/java/lurest-client-api.jar
CLASSPATH += /usr/local/share/java/lurest-service.jar
PROCPATH += /usr/local/share/java/lurest-service.jar
PROCPATH += /usr/local/share/java/lurest-service-proc.jar
EOF
make
sudo make install
</pre>
<p>That will install a number of jars, three Bash scripts, and two Ryu/Python controllers.

<h1>Controller for Corsa</h1>
<p>
If you're setting up the server side of the broker, ensure that a <samp>portslicer.py</samp> Ryu controller is running somewhere.  For example:
<pre>
/usr/bin/ryu-manager --ofp-listen-host 172.31.31.1 --ofp-tcp-listen-port 6556 \
                     --wsapi-host localhost --wsapi-port 8081 \
                     /usr/local/share/dataplane-broker/portslicer.py
</pre>
<p>
It's safe to run as a cronjob; if it's already up, the latest invocation will just quit, and leave the existing one running.  One instance of this controller can operate several Corsa VFCs independently.
<p>
The controller presents two interfaces:
<ul>
<li><samp>172.31.31.1:6556</samp> is the OpenFlow interface, which bridges within the Corsa will contact when set up.  The address you use here appears in configuration below.  Make sure the Corsa has a netns that can access this address.  We'll use <samp>dpb-netns</samp> as an example.
<li><samp>localhost:8081</samp> is the controller's northbound interface, through which the broker expresses intents to the switch.  <samp>localhost</samp> is good enough if the controller and the broker are running in the same machine.
</ul>
<p>Details of these interfaces go in the <samp>ctrl</samp> section of a fabric agent later.
<p>
The advantages of the Corsa fabric include the following:
<ul>
<li>The OpenFlow rules are relatively simple, as Corsa's tunnel attachments handle VLAN tagging and detagging.  As far as the OF switch is concerned, every circuit comes in on its own ofport.
<li>The tunnel attachments also handle bandwidth shaping (on exit from the switch) and metering (on entry).
</ul>
<p>Tunnel attachments are set up through the Corsa's REST-based management interface.

<h1>Controller for OpenFlow 1.3</h1>
<p>
If you're setting up the server side of the broker, ensure that a <samp>tupleslicer.py</samp> Ryu controller is running somewhere.  For example:
<pre>
/usr/bin/ryu-manager --ofp-listen-host 172.31.31.1 --ofp-tcp-listen-port 6556 \
                     --wsapi-host localhost --wsapi-port 8082 \
                     /usr/local/share/dataplane-broker/tupleslicer.py
</pre>
<p>
It's safe to run as a cronjob; if it's already up, the latest invocation will just quit, and leave the existing one running.  One instance of this controller can operate several OpenFlow switches independently.
<p>
The controller presents two interfaces:
<ul>
<li><samp>172.31.31.1:6556</samp> is the OpenFlow interface, which OpenFlow switches will contact when set up.  Make sure the switch's control port can access this address.
<li><samp>localhost:8082</samp> is the controller's northbound interface, through which the broker expresses intents to the switch.
</ul>

<h1>DPB server configuration</h1>

<p>
Create server-side configuration as required in <samp>~/.config/dataplanebroker/server.properties</samp> (a Java 'properties' file, consisting of <samp>dotted.name=value pairs</samp>).  This file must contain a property called <samp>agents</samp> whose value is a comma-separated list of prefixes of other properties.  For example, if it contains “<samp><var>…</var>, foo, <var>…</var></samp>”, then there should also be a set of properties beginning “<samp>foo.</samp>”.
<p>
Here's an example server-side configuration, establishing three persistent switches and one persistent aggregator:
<pre>
&#35;# Two agents are required for each switch, one to implement a generic switch
&#35;# abstraction, and one to adapt it to the physical fabric.
agents=london-switch, london-fabric, \
paris-switch, paris-fabric, \
athens-switch, athens-fabric, \
aggr

aggr.name=aggr
aggr.type=persistent-aggregator
aggr.db.service=jdbc:sqlite:/home/initiate/.local/var/dataplane-broker/aggr.sqlite3

&#35;# The london switch stores its service status in an Sqlite3 database,
&#35;# and operates the fabric provided by the london-fabric agent.
london-switch.name=london
london-switch.type=persistent-switch
london-switch.db.service=jdbc:sqlite:/home/initiate/.local/var/dataplane-broker/london-switch.sqlite3
london-switch.fabric.agent=london-fabric

&#35;# The london fabric operates a single VFC in a Corsa, multiplexing
&#35;# services across it.  It is told how to operate the Corsa's REST
&#35;# management interface, what OpenFlow configuration to apply to the
&#35;# VFC (the controller address, and netns, if it needs to create it),
&#35;# and how to configure the VFC's controller (through a custom REST
&#35;# API).
london-fabric.name=london-fabric
london-fabric.type=corsa-dp2x00-sharedbr
london-fabric.description.prefix=dpb:vc:london:
london-fabric.subtype=openflow
london-fabric.rest.inherit=\#mycorsa.rest
london-fabric.ctrl.inherit=\#mycorsa.ctrl

&#35;# Other switches and fabrics follow a similar pattern.
athens-switch.name=athens
athens-switch.type=persistent-switch
athens-switch.db.service=jdbc:sqlite:/home/initiate/.local/var/dataplane-broker/athens-switch.sqlite3
athens-switch.fabric.agent=athens-fabric

athens-fabric.name=athens-fabric
athens-fabric.type=corsa-dp2x00-sharedbr
athens-fabric.description.prefix=dpb:vc:athens:
athens-fabric.subtype=openflow
athens-fabric.rest.inherit=\#mycorsa.rest
athens-fabric.ctrl.inherit=\#mycorsa.ctrl

paris-switch.name=paris
paris-switch.type=persistent-switch
paris-switch.db.service=jdbc:sqlite:/home/initiate/.local/var/dataplane-broker/paris-switch.sqlite3
paris-switch.fabric.agent=paris-fabric

paris-fabric.name=paris-fabric
paris-fabric.type=corsa-dp2x00-sharedbr
paris-fabric.description.prefix=dpb:vc:paris:
paris-fabric.subtype=openflow
paris-fabric.rest.inherit=\#mycorsa.rest
paris-fabric.ctrl.inherit=\#mycorsa.ctrl

&#35;# These credentials are used to operate a Corsa, and can
&#35;# be shared by multiple simulated switch abstractions.
mycorsa.rest.location=https:\//10.11.12.13/
mycorsa.rest.cert.file=mycorsa-certificate.pem
mycorsa.rest.authz.file=mycorsa-authz
mycorsa.ctrl.host=172.31.31.1
mycorsa.ctrl.port=6556
mycorsa.ctrl.netns=dpb-netns
mycorsa.ctrl.rest.location=http:\//localhost:8081/slicer/
</pre>
<p>
The switch networks each operate a distinct bridge in the same Corsa.  The bridges all connect to the same controller (also running on <samp>beta.sys</samp>).  As the networks access the same Corsa, and the same OpenFlow controller, their control parameters are all inherited from the same configuration, <samp>mycorsa</samp>.  <samp>mycorsa.ctrl</samp> identifies the northbound interface of the OF controller; <samp>mycorsa.rest</samp> identifies the management API of the Corsa.  You will need to enable this on your Corsa, as well as generate credentials to reference from the <samp>rest.cert.file</samp> and <samp>rest.authz.file</samp> fields.  (These filenames are relative to the file referencing them.)
<p>
Each switch network type is <samp>corsa-dp2x00-sharedbr</samp>, meaning a single bridge is created and used for all services.  Each network gives its bridge a distinct description within Corsa management; for example, the <samp>paris</samp> bridge's description begins with <samp>dpb:vc:paris:</samp>.  When the network first contacts the Corsa, it ensures that only one bridge has a description with that prefix.
<p>
In the <samp>corsa-dp2x00-sharedbr</samp> implementation, each active service of <var>N</var> end points manifests as <var>N</var> tunnel attachments to <var>N</var> available ofports of the bridge (which can be viewed on the Corsa with <kbd>show bridge <var>br2</var> tunnel</kbd>).  When <var>N</var>=2, two OpenFlow rules can be seen to exchange traffic from each of these ofports to the other (with <kbd>show openflow flow <var>br2</var></kbd>).  For larger <var>N</var>, an OpenFlow group is created to output to all ofports forming the service (<kbd>show openflow group <var>br2</var></kbd>), learning-switch rules based on this group are added, and unrecognized source MACs are delivered to the controller.
<p>
There is no topology information in the configuration file, as that is set up through other <a href="http://scc-forge.lancaster.ac.uk/javadoc/dataplanebroker-test/uk/ac/london/networks/apps/Commander-method-main/1java$lang$String"><kbd>dpb-client</kbd> commands</a>.  The configuration file only contains management connectivity information.

<h2>SSH authorization<h2>

<p>Access to the server is through SSH.  The script <samp>dpb-ssh-agent</samp> is invoked on the server side, with minimal argumentation, then the client and server engage in a bidirectional exchange of JSON messages.  To allow a user access, take their SSH public key, and append it as a single line to <samp>~/.ssh/authorized_keys</samp> on the server account.  This should be augmented with settings that restrict what the client can invoke:

<pre>
command="/usr/local/bin/dpb-ssh-agent +N -s /tmp/dataplane-broker.sock -m london -m athens -m paris -m aggr",no-port-forwarding,no-agent-forwarding,no-X11-forwarding,no-pty ecdsa-sha2-nistp256 AAAAE2VjZH<var>…</var>f85P+s= foo@example
</pre>

<p>The example above permits the caller to manage and control all four network abstractions (<samp>london</samp>, <samp>athens</samp>, <samp>paris</samp> and <samp>aggr</samp>).  Management operations include the addition and removal of terminals, and the additional, removal and configuration of trunks (for aggregators only).  Control operations include the creation, definition, activation, deactivation and release of services.  Use <samp>-c</samp> to permit only control of a particular network.

<h2>DPB server invocation</h2>

<p>Run the DPB server process through Usmux:

<pre>
usmux -d -B /tmp/dataplane-broker.sock -- dpb-server -s '%s' -L/usr/share/java -L/usr/local/share/java -lsqlite-jdbc -lhttpcore -lhttpclient -lcommons-logging
</pre>

<p>Everything after <samp>--</samp> is the actual DPB server command.  Usmux creates a Unix-domain socket <samp>/tmp/dataplane-broker.sock</samp> (which is also an argument of the SSH configuration above), listens for connections on it, and relays them to the given process.

<p>The <samp>-d</samp> runs in debug mode, i.e., the server process remains in the foreground, and remains connected to the terminal.  Without it, the command exits as soon as the server is established and has forked into the background.

<p>To run as a cronjob, remember to escape the <samp>%</samp> character in the command, as this is special in crontab:

<pre>
*/10 * * * * usmux -B /tmp/dataplane-broker.sock -- dpb-server -s '\%s' -L/usr/share/java -L/usr/local/share/java -lsqlite-jdbc -lhttpcore -lhttpclient -lcommons-logging > /dev/null 2>&amp;1
</pre>

<h1>DPB client configuration</h1>

<p>You'll need an SSH key to get in to remote servers:
<pre>
ssh-keygen -t ecdsa -f ~/.ssh/dpb_ecdsa
</pre>
<p>That creates <samp>dpb_ecdsa</samp> and <samp>dpb_ecdsa.pub</samp> in your SSH directory.  Provide the public key to the server admin to install as noted above.
<p>(You could use your existing key pair, but if you already have unrestricted access to the account, using that key pair will preclude it, as the <samp>authorized_keys</samp> configuration will restrict the user to running only <samp>dpb-ssh-agent</samp>, and only in one particular way.)

<p>Store client-side configuration in <samp>~/.config/dataplanebroker/client.properties</samp>.  The syntax is the same as for the server side.  However, all the agents are just SSH clients into the server account (<samp>initiate@beta.sys</samp> in this example).  For example:

<pre>
&#35;# Define how to SSH to the server host and account.
beta.host=beta.sys
beta.user=initiate
&#35;# (Psst! Not sure if I've implemented ${} yet!)
beta.key-file=${user.home}/.ssh/london-switch_ecdsa

&#35;# Indicate the configuration prefixes of agents to instantiate.
agents=london, paris, athens, aggr

&#35;# Define those agents with corresponding prefixes (e.g., "paris.")...

&#35;# This agent provides access to a remote switch via SSH.
paris.type=ssh-switch
paris.ssh.inherit=\#beta

&#35;# Identify this switch by this name.  This applies to both the
&#35;# command line (i.e., "-n paris"), and the name used at the remote
&#35;# site.
paris.name=paris

&#35;# Other networks are configured similarly...

athens.type=ssh-switch
athens.ssh.inherit=\#beta
athens.name=athens

london.type=ssh-switch
london.ssh.inherit=\#beta
london.name=london

aggr.type=ssh-switch
aggr.ssh.inherit=\#beta
aggr.name=aggr
</pre>

<h1>Client invocation</h1>

<h2>Example topology configuration</h2>

<p>A client with management access can describe the physical topology of the network to the network abstractions.  The following command sets up a simulated chain of switches (<samp>athens</samp>-<samp>paris</samp>-<samp>london</samp>), co-ordinated by <samp>aggr</samp>:
<pre>
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
</pre>

<p>From the aggregator's viewpoint, there are two terminals, <samp>aggr:athens-openstack</samp> and <samp>aggr:london-openstack</samp>.  (The syntax is <samp><var>network</var>:<var>terminal</var></samp>.)  These are mapped to <samp>athens:openstack</samp> and <samp>london:openstack</samp> respectively, which in turn map to the interface configurations <samp>phys.24</samp> (i.e., port 24) and <samp>phys.25</samp>.  This means that users of the aggregator can connect any two or more VLANs ports 24 and 25.

<p>The aggregator is also made aware of two trunks, one between terminals <samp>athens:paris</samp> and <samp>paris:athens</samp>, and one between <samp>london:paris</samp> and <samp>paris:london</samp>.  Both of these run between ports 13 and 29, which have been connected physically with a 10G cable.  5G is allocated to each trunk, as are two non-overlapping VLAN ranges, so we are implementing both trunks on a single link (but just for experimental purposes).

<p>Make sure that the referenced physical ports have the right tunnel mode (usually <samp>ctag</samp>).  Otherwise, tunnel attachment will (quietly!) fail when a service is created.  If this happens, and then you fix the tunnel mode, restarting the broker server will usually cause the tunnels to be (re-)established properly.

<h2>Control operation</h2>

<p>Test connectivity:
<pre>
dpb-client -n aggr dump
</pre>
Create a service:
<pre>
dpb-client -n aggr new
</pre>
Select the service, and define its end points (100Mbps ingress):
<pre>
dpb-client -n aggr -s 1 -b 100 -e athens-openstack:50 -e london-openstack:80 initiate
</pre>
This will plot a spanning tree across the switches, and allocate bandwidth from the physical links.
Activate the service, creating the forwarding rules in the switches:
<pre>
dpb-client -n aggr -s 1 activate
</pre>
De-activate the service:
<pre>
dpb-client -n aggr -s 1 deactivate
</pre>
Activate the service:
<pre>
dpb-client -n aggr -s 1 activate
</pre>
De-activate the service:
<pre>
dpb-client -n aggr -s 1 deactivate
</pre>
Activate the service:
<pre>
dpb-client -n aggr -s 1 activate
</pre>
De-activate the service:
<pre>
dpb-client -n aggr -s 1 deactivate
</pre>
Release the service (resulting in its destruction):
<pre>
dpb-client -n aggr -s 1 release
</pre>
Watch the service for events:
<pre>
dpb-client -n aggr -s 1 watch
</pre>



## Install NBI information

## Install WIM Driver in RO

## DPB RO repo

```
git clone https://github.com/DataPlaneBroker/RO.git
```
