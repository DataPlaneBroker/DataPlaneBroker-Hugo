---
title: 'Installation'
date: 2020-01-13T11:49:39Z
draft: false
weight: 2
summary: "How to install the three broker components: server, SSH client, and Ryu controllers"
---

There are three main components that can be installed together, or separately:

* Server

* SSH client for management and control operations

* Ryu controllers for Corsa and Generic OpenFlow fabrics

They are available as Docker images, and can also be built from source.  The instructions to build from source are annotated with **[Server]**, **[Client]**, **[Controllers]** to indicate which commands to execute depending in which components you're trying to install.

## Docker images

Images are available for the the Corsa Ryu controller, the server, and the client:

```
docker pull simpsonst/dpb-ctrl-corsa
docker pull simpsonst/dpb-server
docker pull simpsonst/dpb-client
```

## Packaged prerequisites

A platform of Ubuntu 18.04 is assumed.  The instructions can surely be adapted for other Linux platforms.

**[Server]** Install Java 9 or later, and other prerequisites:

```
sudo apt-get install -y netcat-openbsd \
	openjdk-11-jdk-headless openjdk-11-jre-headless \
	libjsonp-java libhttpcore-java \
	libhttpclient-java libcommons-logging-java \
	build-essential gawk par subversion git curl zip m4
```

**[Client]** Install Java 9 or later, and other prerequisites:

```
sudo apt-get install -y netcat-openbsd \
	openjdk-11-jdk-headless openjdk-11-jre-headless \
	libjsonp-java build-essential gawk par subversion \
	git curl zip m4
```

**[Controllers]** Install Ryu, and other build tools:

```
sudo apt-get install -y python-ryu build-essential gawk \
	par subversion git
```

**[Client]** **[Server]** **Ubuntu ***16.04*** doesn't have `libjsonp-java`.**  This procedure should install it under `/usr/local/share/java/`:

```
cd /tmp
curl 'http://www.java2s.com/Code/JarDownload/javax.json/javax.json-1.0.jar.zip' -o javax.json-1.0.jar.zip
unzip javax.json-1.0.jar.zip
sudo mkdir -p /usr/local/share/java
sudo mv javax.json-1.0.jar /usr/local/share/java
sudo ln -s javax.json-1.0.jar /usr/local/share/java/javax.json.jar
```

## Pre-built prerequisites

**[Server]** A database connector is also required:

```
sudo mkdir -p /usr/local/share/java
sudo curl 'https://bitbucket.org/xerial/sqlite-jdbc/downloads/sqlite-jdbc-3.23.1.jar' \
	-o /usr/local/share/java/sqlite-jdbc-3.23.1.jar
sudo ln -s sqlite-jdbc-3.23.1.jar /usr/local/share/java/sqlite-jdbc.jar
```

## User-built prerequisites

The DPB and its remaining prerequisites are best checked out from their repositories.  We'll assume you'll keep working copies in a directory such as `~/works/`.

### Installing Binodeps

**[Controllers]** **[Client]** **[Server]** Install [Binodeps](http://www.lancaster.ac.uk/~simpsons/software/pkg-binodeps), which provides enhanced rules for building C and C++ programs using GNU Make, as well as installation rules for scripts and static data:

```
mkdir -p ~/works
cd ~/works
svn co http://scc-forge.lancaster.ac.uk/svn-repos/misc/binodeps/trunk/ binodeps
cd binodeps
make && sudo make install
```

### Installing Jardeps

**[Client]** **[Server]** Install [Jardeps](http://www.lancaster.ac.uk/~simpsons/software/pkg-jardeps), which provides rules for compiling Java with GNU Make:

```
cd ~/works
svn co 'http://scc-forge.lancaster.ac.uk/svn-repos/misc/jardeps/trunk' jardeps
cd jardeps
make && sudo make install
```

### Installing Java REST library

**[Server]** Install Java REST library (noting correct path for `javax.json.jar` if you installed it manually):

```
cd ~/works
svn co 'http://scc-forge.lancaster.ac.uk/svn-repos/utils/lurest/trunk' \
	lurest
cat > lurest-env.mk <<EOF
CLASSPATH += /usr/share/java/httpclient.jar
CLASSPATH += /usr/share/java/httpcore.jar
CLASSPATH += /usr/local/share/java/jardeps-lib.jar
CLASSPATH += /usr/share/java/javax.json.jar
EOF
make && sudo make install
```

[Javadoc](http://scc-forge.lancaster.ac.uk/javadoc/lurest-test/overview-summary) exists for this project.


### Installing DDSLib

**[Server]** Install [DDSLib](http://www.lancaster.ac.uk/~simpsons/software/pkg-smilib), a C library for managing various data structures like linked lists, binary heaps, etc.:

```
cd ~/works
svn co 'http://scc-forge.lancaster.ac.uk/svn-repos/utils/ddslib/branches/stable' ddslib
cd ddslib
cat > ddslib-env.mk <<EOF
CFLAGS += -O2 -g
CFLAGS += -std=gnu11

CPPFLAGS += -D_XOPEN_SOURCE=600
CPPFLAGS += -D_GNU_SOURCE=1
CPPFLAGS += -pedantic -Wall -W -Wno-unused-parameter
CPPFLAGS += -Wno-missing-field-initializers

CXXFLAGS += -O2 -g
CXXFLAGS += -std=gnu++11
EOF
make && sudo make install
```


### Installing Reactor

**[Server]** Install [Reactor](http://www.lancaster.ac.uk/~simpsons/software/pkg-react), a library for event-driven programming in C:

```
cd ~/works
svn co 'http://scc-forge.lancaster.ac.uk/svn-repos/utils/react/trunk' react
cd react
cat > react-env.mk <<EOF
ENABLE_CXX=no
CFLAGS += -O2 -g
CFLAGS += -std=gnu11

CPPFLAGS += -D_XOPEN_SOURCE=600
CPPFLAGS += -D_GNU_SOURCE=1
CPPFLAGS += -pedantic -Wall -W -Wno-unused-parameter
CPPFLAGS += -Wno-missing-field-initializers

CXXFLAGS += -O2 -g
CXXFLAGS += -std=gnu++11
EOF
make && sudo make install
```


### Installing Usmux

**[Server]** Install [Usmux](http://www.lancaster.ac.uk/~simpsons/software/pkg-usmux), which allows a Java process to receive connections over a Unix-domain socket, and fork into the background.  The DPB server process makes itself available over such a socket.

```
cd ~/works
svn co 'http://scc-forge.lancaster.ac.uk/svn-repos/misc/usmux/branches/export' usmux
cd usmux
cat > usmux-env.mk <<EOF
CFLAGS += -O2 -g
CFLAGS += -std=gnu11
CPPFLAGS += -D_XOPEN_SOURCE=600
CPPFLAGS += -D_GNU_SOURCE=1
CPPFLAGS += -pedantic -Wall -W -Wno-unused-parameter
CPPFLAGS += -Wno-missing-field-initializers
EOF
make && sudo make install
```


## Installing DPB

**[Server]** These commands also install the controllers and the client as a side-effect.  Note the correct path for `javax.json.jar` if you installed it manually.

```
cd ~/works
git clone git@github.com:DataPlaneBroker/DPB.git
cd DPB
cat > dataplanebroker-env.mk <<EOF
CLASSPATH += /usr/share/java/junit4.jar
CLASSPATH += /usr/share/java/javax.json.jar
CLASSPATH += /usr/share/java/httpcore.jar
CLASSPATH += /usr/share/java/httpclient.jar
CLASSPATH += /usr/share/java/commons-logging.jar
CLASSPATH += /usr/local/share/java/jardeps-lib.jar
CLASSPATH += /usr/local/share/java/lurest-client-api.jar

CLASSPATH += /usr/local/share/java/usmux_session.jar
CLASSPATH += /usr/local/share/java/lurest-server-api.jar
CLASSPATH += /usr/local/share/java/lurest-service.jar
PROCPATH += /usr/local/share/java/lurest-service.jar
PROCPATH += /usr/local/share/java/lurest-service-proc.jar
EOF
make && sudo make install
```

**[Client]** These commands also install the controllers as a side-effect.  Note the correct path for `javax.json.jar` if you installed it manually.

```
cd ~/works
git clone git@github.com:DataPlaneBroker/DPB.git
cd DPB
cat > dataplanebroker-env.mk <<EOF
CLASSPATH += /usr/share/java/junit4.jar
CLASSPATH += /usr/share/java/javax.json.jar
CLASSPATH += /usr/share/java/httpcore.jar
CLASSPATH += /usr/share/java/httpclient.jar
CLASSPATH += /usr/share/java/commons-logging.jar
CLASSPATH += /usr/local/share/java/jardeps-lib.jar
CLASSPATH += /usr/local/share/java/lurest-client-api.jar
EOF
make client && sudo make install-client
```

**[Controllers]**

```
cd ~/works
git clone git@github.com:DataPlaneBroker/DPB.git
cd DPB
sudo make install-ctrl
```



---

## Install NBI information

## Install WIM Driver in RO

## DPB RO repo

```
git clone https://github.com/DataPlaneBroker/RO.git
```

Erm, probably not to be done any more, as it's integrated into OSM now, right?
