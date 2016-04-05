<!--- master only -->
> ![warning](images/warning.png) This document applies to the HEAD of the calico-mesos-deployments source tree.
>
> View the calico-mesos-deployments documentation for the latest release [here](https://github.com/projectcalico/calico-mesos-deployments/blob/0.27.0%2B2/README.md).
<!--- else
> You are viewing the calico-mesos-deployments documentation for release **release**.
<!--- end of master only -->

# Vagrant Deployed Mesos Cluster with Calico using Docker Containerizer
This guide will start a running Mesos cluster (master and two agents) with Calico Networking using a simple `vagrant up`.
These machines will be configured to use the Docker containerizer for launching tasks.

## Prerequisites
This guide requires a host machine with:

 * [VirtualBox][virtualbox] to host the virtual machines.
 * [Vagrant][vagrant] to install and configure the machines in Virtual Box.
 * [Git](git-scm.com)

## Getting Started
1. First, clone this repository and change to the Mesos vagrant directory:

  ```
  git clone https://github.com/projectcalico/calico-containers.git
  cd calico-containers/docs/mesos/vagrant-centos
  ```

2. Then launch the Vagrant demo:
  ```
  vagrant up
  ```

You can log into each machine by running:
```
vagrant ssh <HOSTNAME>
```

That's it! Your Mesos Cluster is ready to use!

## Next steps

To view an interesting demo with a network policy visualizer,
you can run through the [Calico Mesos Stars Demo](stars-demo/README.md).

To learn how to configure tasks using the Docker Containerizer with
Calico networking, checkout out our [Docker Containerizer Usage Guide]
(./UsageGuideDockerContainerizer).

## Virtual Machines Info

The install virtual machines will be running with the following config:

### Master
All services on Master are installed .
 * **OS**: `Centos`
 * **Hostname**: `calico-mesos-01`
 * **IP**: `172.24.197.101`
 * **Running Services**:
   * `mesos-master` (RPM)
   * `docker`
 * **Docker Containers**
   * `etcd` (Docker)
   * `zookeeper` (Docker)
   * `marathon` (Docker)
   * `marathon load balancer` (Docker)
   * `calico-node`
   * `calico-libnetwork`

### Agents
Mesos is installed with Netmodules using the Netmodules RPM bundle, and Calico is added and configured using the calico-mesos.rpm provided in this repository. Once both are installed, the agent will match the following specification:
 * **OS**: `Centos`
 * **Hostnames**: `calico-mesos-02`, `calico-mesos-03`
 * **IP**: `172.24.197.102`, `172.24.197.103`
 * **Running Services**:
   * `mesos-agent` (RPM)
   * `docker`
 * **Docker Containers**:
   * `calico-node`
   * `calico-libnetwork`

## Adding More Agents (Optional)
You can modify the script to use multiple agents. To do this, modify the `num_instances` variable
in the `Vagrantfile` to be greater than `2`.  The first instance created is the master instance, every 
additional instance will be an agent instance.

Every agent instance will take similar form to the agent instance above:

 * **OS**: `Centos`
 * **Hostname**: `calico-mesos-0X`
 * **IP**: `172.24.197.10X`
 * **Running Services**:
   * `mesos-agent`
 * **Docker Containers**:
   * `calico-node`
   * `calico-libnetwork`

where `X` is the instance number.

If you've already run the vagrant script but want to add more Agents, just
change the `num_instances` variable then run `vagrant up` again.  Your
existing VMs will remain installed and ready.

[virtualbox]: https://www.virtualbox.org/
[vagrant]: https://www.vagrantup.com/
[![Analytics](https://calico-ga-beacon.appspot.com/UA-52125893-3/calico-containers/docs/mesos/DockerizedVagrant.md?pixel)](https://github.com/igrigorik/ga-beacon)
