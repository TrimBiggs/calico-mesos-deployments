<!--- master only -->
> ![warning](images/warning.png) This document applies to the HEAD of the calico-mesos-deployments source tree.
>
> View the calico-mesos-deployments documentation for the latest release [here](https://github.com/projectcalico/calico-mesos-deployments/blob/0.27.0%2B2/README.md).
<!--- else
> You are viewing the calico-mesos-deployments documentation for release **release**.
<!--- end of master only -->

# Manually Install Calico and Mesos on Agent
This tutorial will walk you through installing Mesos, Netmodules, and Calico onto a Centos host to create a Mesos agent compatible with Calico.

## Prerequisites
### Zookeeper, etcd, and Marathon
This guide assumes you have already set up these highly available services. If you do not have these services running, follow the [Cluster Preperation Guide](MesosClusterPreparation.md) for information on the fastest way to set these services up before continuing.

### Mesos-Master
A Mesos cluster running Calico does not require any modifications to the master. Simply follow [Mesosphere's Mesos Install Guide](https://mesosphere.com/downloads/) to configure your master.

### Set & verify fully qualified domain name
These instructions assume each host can reach other hosts using their fully qualified domain names (FQDN).  To check the FQDN on a host use

```
hostname -f
```
Then attempt to ping that name from other hosts.

It is also important that Calico and Mesos have the same view of the (non-fully-qualified) hostname.  Ensure that the value returned by `hostname` is unique for each host in your cluster.

### Docker
Calico's core services are run in a docker container, so we'll need Docker installed on every Agent in the cluster.
[Follow Docker's Centos installation guide](https://docs.docker.com/engine/installation/centos/) for information on how to get Docker installed.


## 1. Configure firewall
You will either need to configure the firewalls on each node in your cluster (recommended) to allow access to the cluster services or disable them completely.  This section contains configuration examples for `firewalld`.  If you use a different firewall, check your documentation for how to open the listed ports.

Agent (compute/slave) nodes require

| Service Name | Port/protocol     |
|--------------|-------------------|
| BIRD (BGP)   | 179/tcp           |
| mesos-agent  | 5051/tcp          |

Example `firewalld` config:

```
sudo firewall-cmd --zone=public --add-port=179/tcp --permanent
sudo firewall-cmd --zone=public --add-port=5051/tcp --permanent
sudo systemctl restart firewalld
```

## 2. Download the Calico Mesos Plugin
The Calico-Mesos plugin is available for download from the [calico-mesos repository releases](https://github.com/projectcalico/calico-mesos/releases). In this example, we will install the binary to the `/calico` directory.

```
wget https://github.com/projectcalico/calico-mesos/releases/download/v0.1.5/calico_mesos
chmod +x calico_mesos
sudo mkdir /calico
sudo mv calico_mesos /calico/calico_mesos
```
## 3. Create the modules.json Configuration File
To enable Calico networking in Mesos, you must create a `modules.json` file. When provided to the Mesos Agent process, this file will connect Mesos with the Net-Modules libraries as well as the Calico networking plugin, thus allowing Calico to 
receive networking events from Mesos.

```
wget https://raw.githubusercontent.com/projectcalico/calico-mesos-deployments/master/packages/sources/modules.json
sudo mv modules.json /calico/modules.json
```

## 4. Run Calico Node
The last Calico component required for Calico networking in Mesos is `calico-node`, a Docker image containing Calico's core routing processes.
 
`calico-node` can easily be launched via `calicoctl`, Calico's command line tool. When doing so, we must point `calicoctl` to our running instance of etcd, by setting the `ECTD_AUTHORITY` environment variable.

> Follow our [Mesos Cluster Preparation guide](MesosClusterPreparation.md#Install) if you do not already have an instance of etcd running to walk through installing etcd with Docker.

```
sudo yum install -y wget
wget https://github.com/projectcalico/calico-containers/releases/download/v0.9.0/calicoctl
chmod +x calicoctl
sudo ETCD_AUTHORITY=<IP of host with etcd>:4001 ./calicoctl node
```

## 5. Install Mesos / Netmodules Dependencies
Netmodules and Mesos both make use of the `protobuf`, `boost`, and `glog` libraries. To function correctly, Mesos and Netmodules must be built with identical compilations of these libraries. A standard Mesos installation will include bundled versions, so we'll compile Mesos with unbundled versions to ensure that netmodules is using precisely the same library as Mesos. First, download the libraries:

```
sudo yum update
sudo yum groupinstall -y 'Development Tools'

sudo yum update
sudo yum install -y wget docker git autoconf automake libcurl-devel \
apr-devel subversion-devel java java-devel protobuf-devel protobuf-python \
boost-devel zlib-devel maven libapr-devel cyrus-sasl-md5 python-devel

# Modify this value if your java home is different.
export JAVA_HOME=/usr/lib/jvm/java
```

At the time of this writing, the `glog-devel` rpm package (available after installing and updating `epel-release`) does not satisfy Mesos' glog dependency.

Run the following to manually compile and install glog v0.3.3:
```
git clone https://github.com/google/glog.git -b v0.3.3
cd glog
sudo ./configure --prefix=/usr --libdir=/usr/lib64 && sudo make && sudo make install
```
Next, install the picojson headers and protobuf.jar file:

```
sudo wget https://raw.githubusercontent.com/kazuho/picojson/v1.3.0/picojson.h -O /usr/local/include/picojson.h

sudo wget http://search.maven.org/remotecontent?filepath=com/google/protobuf/protobuf-java/2.5.0/protobuf-java-2.5.0.jar -O /usr/share/java/protobuf.jar
```

## 6. Build and Install Mesos
Next we'll follow the standard Mesos installation instructions, but pass a few flags to configure to use our installed libraries instead of the mesos bundled ones:

```
# Download Mesos source
git clone git://git.apache.org/mesos.git -b 0.26.0
cd mesos

# Configure and build.
./bootstrap
mkdir build
cd build
../configure --with-protobuf=/usr --with-boost=/usr --with-glog=/usr
make
sudo make install
```

## 7. Build and Install Netmodules
We install netmodules as a plugin to allow Calico to interface with Mesos.

```
# Download netmodules source
git clone https://github.com/mesosphere/net-modules.git
cd net-modules/isolator

# Configure and build
./bootstrap
mkdir build
cd build
../configure --with-mesos=/usr/local --with-protobuf=/usr
make
sudo make install
```

## 8. Launch Mesos-Slave

```
sudo ETCD_AUTHORITY=<ETCD-IP:PORT> /usr/local/sbin/mesos-slave \
--master=<MASTER-IP:PORT> \
--modules=file:///calico/modules.json \
--isolation=com_mesosphere_mesos_NetworkIsolator \
--hooks=com_mesosphere_mesos_NetworkHook
```
We provide the `ETCD_AUTHORITY` environment variable here to allow the  `calico_mesos` plugin to function properly when called by `mesos-slave`. Be sure to replace it with the address of your running etcd server.

## Next steps
See [Our Guide on Using Calico-Mesos](UsingCalicoMesos.md) for info on how to test your cluster and start launching tasks networked with Calico.

[![Analytics](https://calico-ga-beacon.appspot.com/UA-52125893-3/calico-containers/docs/mesos/ManualInstallCalicoMesos.md?pixel)](https://github.com/igrigorik/ga-beacon)
