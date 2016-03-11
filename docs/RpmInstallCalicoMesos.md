# RPM Installation of Mesos with Calico
This tutorial shows how to configure a Mesos Agent able to launch tasks networked with calico networking.
At the completion of this guide, you will have a Mesos Agent ready to launch tasks with their own Calico assigned IP Addresses using the following services and configuration files:
- mesos
- net-modules
- `calico_mesos` plugin binary
- `modules.json`, JSON file which points mesos to the location of `net-modules` and points `net-modules` to the `calico-mesos` plugin
- `calicoctl`, a command line tool for easily launching the calico-node service
- `calico-mesos.service`, a systemd service to ensure calico is always running

>Note: These RPMs do not serve as an official calico or mesos installation option, and will not receive a supported upgrade story for the future. They merely serve as a more automatic alternative to performing the steps in the [Manually Installing Calico + Mesos + Netmodules](ManualInstallCalicoMesos.md) guide.

## Prerequisites
### Zookeeper and etcd
Follow the [Mesos Master and Cluster Preparation Guide](MesosClusterPreparation.md) for information on configuring these services.

### Mesos Master
Adding Calico to your Mesos Cluster doesn't require any modifications to the Mesos Master. Therefore you can follow Mesosphere's  [Mesos Install Guide](https://open.mesosphere.com/getting-started/install/#master-setup) using their official RPM to set up your Master.

## Getting Started
We will now install Mesos, Netmodules, and Calico on an Agent.

### Download and Install the RPMs
1. Extra Packages for Enterprise Linux (EPEL) must be installed before installing Mesos + Net-Modules. You can download this package by calling:
  ```
  sudo yum install -y epel-release
  sudo yum update
  ```

2. Install the custom Mesos + Netmodules RPMs:
  ```
  curl -O https://github.com/projectcalico/calico-mesos-deployments/releases/download/0.27.0%2B2/mesos-netmodules-rpms.tar
  tar -xvf mesos-netmodules-rpms.tar
  sudo yum install -y mesos-netmodules-rpms/*.rpm
  ```

3. Install the Calico-Mesos RPM:
  ```
  curl -O https://github.com/projectcalico/calico-mesos-deployments/releases/download/0.27.0%2B2/calico-mesos.rpm
  sudo yum install -y ./calico-mesos.rpm
  ```

### Start Calico
A systemd unit file has been provided to start the Calico processes needed by the `calico_mesos` plugin binary. We'll need to configure a few environment variables to tell Calico and Mesos where etcd and zookeeper are running.

> If you do not already have zookeeper and etcd running, follow the [Mesos Cluster Preparation guide](MesosClusterPreparation.md#install-zookeeper-and-etcd).

1. Open `/etc/default/mesos-slave` set the `ETCD_AUTHORITY` and `MASTER` variables to the correct values.  Your file should now look like this:
    ```
    MASTER=zk://<ZOOKEEPER_IP>:2181/mesos
    ETCD_AUTHORITY=<ETCD_IP>:4001
    ```

2. Now start the services:
    ```
    sudo systemctl start calico-mesos.service
    sudo systemctl start mesos-slave.service
    ```

3. Check that Calico and Mesos are both running:
    ```
    sudo systemctl status calico-mesos.service
    sudo systemctl status mesos-slave.service
    ```

## Next steps
See [Our Guide on Using Calico-Mesos](UsingCalicoMesos.md) for info on how to test your cluster and start launching tasks networked with Calico.

[calico-mesos]: https://github.com/projectcalico/calico-mesos/releases/latest

[![Analytics](https://calico-ga-beacon.appspot.com/UA-52125893-3/calico-containers/docs/mesos/RpmInstallCalicoMesos.md?pixel)](https://github.com/igrigorik/ga-beacon)
