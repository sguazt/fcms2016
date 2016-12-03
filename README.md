fcms2016
========

This is a meta-project for the code used in the experimental evaluation presented in the article:

> Cosimo Anglano, Massimo Canonico and Marco Guazzone.
>
> *FCMS: a Fuzzy Controller for CPU and Memory Consolidation under SLA Constraints*
>
> Concurrency and Computation: Practice and Experience, In Press, 2016.
>
> doi:[10.1002/cpe.3968](http://dx.doi.org/10.1002/cpe.3968).

Please, cite this project as follows (BibTeX format):

    @ARTICLE{CPE:CPE3968,
        author = {Cosimo Anglano and Massimo Canonico and Marco Guazzone},
        title = {{FCMS}: A Fuzzy Controller for {CPU} and Memory Consolidation under {SLA} Constraints},
        journal = {Concurrency and Computation: Practice and Experience},
        year = {2016},
        keywords = {cloud computing, feedback control, fuzzy control, resource management, server consolidation, virtualized cloud applications},
        url = {http://dx.doi.org/10.1002/cpe.3968},
        doi = {10.1002/cpe.3968},
        issn = {1532-0634},
		note = {In Press},
    }


Introduction
------------

The project is composed by two macro-modules:

* *server-side module* contains code to be run on the server-side (e.g., inside a virtual machine (VM)). The module is composed by:

	* `server/apache-cassandra-2.0.17-bin.tar.gz`: the archive containing the binaries of the [Apache Cassandra](http://cassandra.apache.org) (v2.0.17)
	* `server/olio-dev.zip`: the archive containing the [patched sources](http://github.com/sguazt/olio) of the [Apache Olio](http://incubator.apache.org/projects/olio.html) application (v0.2)
	* `server/redis-2.8.24.tar.gz`: the archive containing the sources of the [Redis](http://redis.io) (v2.8.24)
	* `server/RUBBoS-dev.zip`: the archive containing the [patched sources](http://github.com/sguazt/RUBBoS) of the [OW2 RUBBoS](http://jmob.ow2.org/rubbos.html) application (v1.2.2)
	* `server/RUBiS-dev.zip`: the archive containing the [patched sources](http://github.com/sguazt/RUBiS) of the [OW2 RUBiS](http://rubis.ow2.org) application (v1.4.3)

* *client-side module* contains code to be run on the client-side. The module is composed by:

	* `client/boost-ublasx-1.1.1.zip`: the archive containing the sources of the [Boost.uBLASx](http://github.com/sguazt/boost-ublasx) (v1.1.1)
	* `client/dcsxx-commons-2.1.1.zip`: the archive containing the sources of the [dcsxx-commons](http://github.com/sguazt/dcsxx-commons) (v2.1.1)
	* `client/dcsxx-control-2.1.1.zip`: the archive containing the sources of the [dcsxx-control](http://github.com/sguazt/dcsxx-control) (v2.1.1)
	* `client/dcsxx-sysid-1.0.0.zip`: the archive containing the sources of the [dcsxx-sysid](http://github.com/sguazt/dcsxx-sysid) (v1.0.0)
	* `client/dcsxx-testbed-4.1.0.zip`: the archive containing the sources of the [dcsxx-testbed](http://github.com/sguazt/dcsxx-testbed) (v4.1.0)
	* `client/fuzzylitex-1.1.1.zip`: the archive containing the sources of the [fuzzylitex](http://github.com/sguazt/fuzzylitex) (v1.1.1)
	* `client/rain-workload-toolkit-dev.zip`: the archive containing the sources of the [RAIN](http://github.com/yungsters/rain-workload-toolkit) workload toolkit (development version)

For a more updated version of the above components, see the related project Web sites.


Server-side Setup
-----------------

In your host operating system, you must install the [Xen](http://www.xen.org) hypervisor.
Furthermore, each tier of the applications you plan to run, should be installed in a separate VM.
Finally, to enable remote communications between the Xen hypervisor and client-side components, you need to install the [libvirt](http://libvirt.org) virtualization API and run the `libvirtd` daemon.

In our experiments, we used the [Fedora](http://fedoraproject.org) 21 Linux distribution as the host operating system.
For a guide on how to install and setup Xen on Fedora you can, for instance, refer to the [Fedora Host Installation](http://wiki.xen.org/wiki/Fedora_Host_Installation) page of the Xen's wiki site.

For each application tier, we setup a dedicated VM, that is:

* For Cassandra, we created only one VM.
* For Redis, we created only one VM.
* For Olio, we created two VMs, one for the Web tier and another one for the DB tier.
* For RUBBoS, we created two VMs, one for the Web tier and another one for the DB tier.
* For RUBiS, we created two VMs, one for the Web tier and another one for the DB tier.

For each of the above VM, we used the [CentOS](http://www.centos.org) 7.2 Linux distribution as the guest operating system.

### Prerequisites

* [libvirt](http://libvirt.org) virtualization API library (v1.2.9 or newer)
* [Xen](http://www.xen.org) virtualization platform (v4.4 or newer)

### Libvirt Daemon Setup

The `libvirtd` daemon is to be started on the host system in order to enable remote communications between the client-side components and the (server-side) VMM hypervisor (Xen in our case).

In the following we setup *libvirt* in order to accept remote communications by using secure TLS connections on the public TCP/IP port.
When we refer to the *server machine* we mean the machine that accepts remote requests and where a `libvirtd` daemon instance is running (i.e., the host system).
Instead, when we refer to the *client machine* we mean the machine from which you issue remote requests (i.e., the machine that runs the client-side components).

In the rest of this section, we assume a Red Hat Linux like operating system.

1. On the *server machine*, it is necessary to start the `libvirtd` server in listening mode by running it with the `--listen` option or by editing the `/etc/sysconfig/libvirtd` file to add the `LIBVIRTD_ARGS="--listen"` line, in order to cause `libvirtd` to come up in listening mode whenever it is started.
2. Also, in order to accept remote connections, you need to open in your firewall the `libvirtd` port, which is usually `16514` (check your `/etc/libvirt/libvirtd.conf` file), for the TCP protocol.
3. Setup a Certificate Authority (CA), for instance, by using the `certtool` utility from the *GnuTLS* library.
   After this step, you have two files, say:

	* `cakey.pem`: your CA's (secret) private key
	* `cacert.pem`: your CA's (public) certificate
4. Install the file `cacert.pem` on both the *client* and *server machines* to let them know that they can trust certificates issued by your CA.
   The usual installation directory for `cacert.pem` is `/etc/pki/CA`
5. Issue servers certificates.
   In the *server machine*, you need to issue a certificate with the *X.509 CommonName* (CN) field set to the host name of the server.
   The CN must match the host name that clients will use to connect to the server.
   After this step, you have two files, say:

	* `serverkey.pem`: is the server's private key
	* `servercert.pem`: is the server's certificate

   that have to be installed on the server.
   Also note that the `serverkey.pem` file **must** have file permissions set to `600`.
   The usual installation directory for `serverkey.pem` is `/etc/pki/libvirt/private`.
   The usual installation directory for `servercert.pem` is `/etc/pki/libvirt`.
6. Issue clients certificates.
   In the *client machine* you need to issue a certificate with the *X.509 Distinguished Name* (DN) set to a suitable name (e.g., the client name).
   Also, make sure the server host name is recognized by your client system (e.g., put it in the `/etc/hosts` file).
   After this step, you have two files, say:

	* `clientkey.pem`: is the client's private key
	* `clientcert.pem`: is the client's certificate

   that have to be installed on the client.
   The usual installation directory for `clientkey.pem` is `/etc/pki/libvirt/private`.
   The usual installation directory for `clientcert.pem` is `/etc/pki/libvirt`.

### Cassandra Setup

The *Apache Cassandra* version included in this distribution does not need to be compiled since it already comes in a binary form.

1. Create one VM
2. Copy the files `server/apache-cassandra-2.0.17-bin.tar.gz` and `server/meminfo-srv.py` inside the VM.
3. Log into the VM.
4. Uncompress the file `apache-cassandra-2.0.17-bin.tar.gz`
5. Follow instructions at [Datastax](http://docs.datastax.com/en/cassandra/2.0/)
6. Run `meminfo-srv.py` in a separate `screen` session.

### Olio Setup

1. Create two VMs, one for the Web tier and another one for the DB tier.
2. Copy the file `server/olio-dev.zip` and `server/meminfo-srv.py` inside each VM.
3. Unzip the file `olio-dev.zip`.
4. Follow building instructions (for the tier running inside the VM) at `olio/docs/php_setup.html`.
5. Run `meminfo-srv.py` in a separate `screen` session.

### Redis Setup

1. Create one VM
2. Copy the file `server/redis-2.8.24.tar.gz` and `server/meminfo-srv.py` inside the VM.
3. Log into the VM.
4. Uncompress the file `redis-2.8.24.tar.gz`.
5. Follow building instructions at `https://github.com/antirez/redis`.
6. Run `meminfo-srv.py` in a separate `screen` session.

### RUBBoS Setup

1. Create two VMs, one for the Web tier and another one for the DB tier.
2. Copy the file `server/RUBBoS-dev.zip` and `server/meminfo-srv.py` inside each VM.
3. Log into each VM.
4. Unzip the file `RUBBoS-dev.zip`.
5. Follow building instructions (for the tier running inside the VM) at `RUBBoS/README.md`, related to the PHP incarnation of RUBBoS.
6. Run `meminfo-srv.py` in a separate `screen` session.

### RUBiS Setup

1. Create two VMs, one for the Web tier and another one for the DB tier.
2. Copy the file `server/RUBiS-dev.zip` and `server/meminfo-srv.py` inside each VM.
3. Log into each VM.
4. Unzip the file `RUBiS-dev.zip`.
5. Follow building instructions (for the tier running inside the VM) at `RUBiS/README.md`, related to the PHP incarnation of RUBiS.
6. Run `meminfo-srv.py` in a separate `screen` session.


Client-side Setup
-----------------

### Prerequisites

* A modern C++98 compiler (e.g., GCC v4.9 or newer is fine)
* The GNU make tool
* [Apache Ant](http://ant.apache.org) (v1.9 or newer)
* [Boost](http://boost.org) C++ libraries (v1.60 or newer)
* [Boost.Numeric Bindings](http://svn.boost.org/svn/boost/sandbox/numeric_bindings) library (v2 or newer)
* [fuzzylite](http://www.fuzzylite.com) fuzzy logic control library (v5 or newer)
* [jsoncpp](https://github.com/open-source-parsers/jsoncpp) C++ library for interacting with JSON (v0.6 or newer)
* [LAPACK](http://www.netlib.org/lapack/) Linear Algebra PACKage (v3.5 or newer)
* [libvirt](http://libvirt.org) virtualization API library (v1.2 or newer)
* [Oracle Java SE](http://www.oracle.com/technetwork/java/javase/index.html) SDK (v7 or newer)

Also, for a more detailed list of requirements, see the documentation of the various included sub-projects.

### RAIN Setup

1. Unzip file `files/rain-workload-toolkit-dev.zip`
2. Change current directory into `rain-workload-toolkit`
3. Run `ant package`
4. For Cassandra workload driver:

	1. Edit `config/rain.config.cassandra.json` and `config/profiles.config.olio.json` (see `rain-workload-toolkit/src/radlab/rain/workload/cassandra/README.md`, for more information)
	2. Run `ant package-cassandra`
5. For Olio workload driver:

	1. Edit `config/rain.config.olio.json` and `config/profiles.config.olio.json` (see `rain-workload-toolkit/src/radlab/rain/workload/olio/README.md`, for more information)
	2. Run `ant package-olio`
6. For Redis workload driver:

	1. Edit `config/rain.config.redis.json` and `config/profiles.config.redis.json` (see `rain-workload-toolkit/src/radlab/rain/workload/redis/README.md`, for more information)
	2. Run `ant package-redis`

7. For RUBBoS workload driver:

	1. Edit `config/rain.config.rubbos.json` and `config/profiles.config.rubbos.json` (see `rain-workload-toolkit/src/radlab/rain/workload/rubbos/README.md`, for more information)
	2. Run `ant package-rubbos`

8. For RUBiS workload driver:

	1. Edit `config/rain.config.rubis.json` and `config/profiles.config.rubis.json` (see `rain-workload-toolkit/src/radlab/rain/workload/rubis/README.md`, for more information)
	2. Run `ant package-rubis`

### dcsxx-testbed Setup

1. Unzip file `files/dcsxx-testbed-4.1.0.zip`
2. Follow instructions in `dcsxx-testbed/README.md`


Bugs and Contributions
----------------------

Bug notification and patches are always welcomed.
Also, other type of contributions (e.g., new features or improvement) are welcomed, as well.

Please, note that, since this is only a meta-project (i.e., a container for other sub-projects), for feedback and contributions you are asked to refer to the specific sub-project.
