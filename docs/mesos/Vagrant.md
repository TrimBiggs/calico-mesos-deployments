<!--- master only -->
> ![warning](images/warning.png) This document applies to the HEAD of the calico-mesos-deployments source tree.
>
> View the calico-mesos-deployments documentation for the latest release [here](https://github.com/projectcalico/calico-mesos-deployments/blob/0.27.0%2B2/README.md).
<!--- else
> You are viewing the calico-mesos-deployments documentation for release **release**.
<!--- end of master only -->

# Vagrant Deployed Mesos Cluster with Calico
This guide will start a running Mesos cluster (master and two agents) with Calico Networking using a simple `vagrant up`.
These machines will be configured to use the Docker containerizer for launching tasks.

> NOTE: The vagrant script and guide do not currently support the Unified Containerizer.
> This guide should only be used in conjunction with the Docker Containerizer.

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

That's it! Your Mesos Cluster is ready to use!

## Log in to Vagrant machines

To connect to your servers:

####Linux/Mac OS X
Run:

	vagrant ssh <hostname>

####Windows
Follow instructions from https://github.com/nickryand/vagrant-multi-putty.

Then, run:

	vagrant putty <hostname>

## Next steps

With your cluster deployed, you have everything in place to run the
[Calico Mesos Stars Demo](stars-demo/README.md), an interesting network
policy visualizer demo that shows how Calico can secure your cluster.

To learn more generally how to configure tasks using the Docker Containerizer
with Calico networking, check out our [Docker Containerizer Usage Guide]
(./UsageGuideDockerContainerizer).

## Virtual Machines Info

The install virtual machines will be running with the following config:

```
.-----------------------------------------------------------------------------------.
| Machine Type | OS     | Hostname        | IP Address     | Services               |
|--------------|--------|-----------------|----------------|------------------------|
| Master       | Centos | calico-mesos-01 | 172.24.197.101 | mesos-master           |
|              |        |                 |                | etcd                   |
|              |        |                 |                | docker                 |
|              |        |                 |                | zookeeper              |
|              |        |                 |                | marathon               |
|              |        |                 |                | marathon load-balancer |
|              |        |                 |                | calico-node            |
|              |        |                 |                | calico-libnetwork      |
|--------------|--------|-----------------|----------------|------------------------|
| Agents       | Centos | calico-mesos-02 | 172.24.197.102 | mesos-agent            |
|              |        | calico-mesos-03 | 172.24.197.103 | docker                 |
|              |        |                 |                | calico-node            |
|              |        |                 |                | calico-libnetwork      |
'-----------------------------------------------------------------------------------'
```

Note that Calico is installed on the Agents so that tasks are automatically
networked by Calico.  Calico is also installed on the Master to provide an IP
address to the Marathon load-balancer, which is used in the stars demo.

## Adding More Agents (Optional)
You can modify the script to use multiple agents. To do this, modify the `num_instances`
variable in the `Vagrantfile`.  The variable is set to `3`, where the first machine is the
master and every other machine is an agent.  If you'd like four agents, set `num_instances`
to be `5`.

Every agent instance will take similar form to the agent instances in the table above.

The `hostname` and `IP address` of the machines are generated in the vagrant script,
so additional machines will take these values in the form:

	Hostname:   `calico-mesos-0X`
	IP address: `172.24.197.10X`

where `X` is the instance number.

If you've already run the vagrant script but want to add more Agents, just
change the `num_instances` variable then run `vagrant up` again.  Your
existing VMs will remain installed and ready.

[virtualbox]: https://www.virtualbox.org/
[vagrant]: https://www.vagrantup.com/
[![Analytics](https://calico-ga-beacon.appspot.com/UA-52125893-3/calico-containers/docs/mesos/DockerizedVagrant.md?pixel)](https://github.com/igrigorik/ga-beacon)
