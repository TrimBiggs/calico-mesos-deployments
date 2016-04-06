<!--- master only -->
> ![warning](../images/warning.png) This document applies to the HEAD of the calico-containers source tree.
>
> View the calico-containers documentation for the latest release [here](https://github.com/projectcalico/calico-containers/blob/v0.17.0/README.md).
<!--- else
> You are viewing the calico-containers documentation for release **release**.
<!--- end of master only -->

#TODO: Point at new install docs for different containerizers/stars demo/prereqs

# Mesos with Calico networking
Calico can be used as a network plugin for Mesos both for the Docker
Containerizer and the Unified Containerizer.

### Docker Containerizer
Calico with the Docker Containerizer uses a Calico network and IPAM 
driver that hooks directly into the Docker networking infrastructure.
Docker networks using the Calico plugin are created up-front, and Mesos 
can then be used to launch Docker containers using these networks.  Each
network is associated with a single Calico profile.  Fine grained policy
can be modified using the `calicoctl profile` commands.

### Unified Containerizer
Calico with the Unified Containerizer uses the [Calico Mesos plugin]
(https://github.com/projectcalico/calico-mesos) to configure
networking for a Mesos agent that is running [net-modules]
(https://github.com/mesosphere/net-modules). Networks are
specified as net-groups when launching a Mesos task.  A Calico
profile is automatically created for each net-group (if it doesn't
already exist), defaulting to allow communication between containers
using the same net-group.  Fine grained policy can be modified using
the `calicoctl profile` commands.

## Cluster Configuration Requirements
The installation requirements to use Calico networking are different
depending on whether you are using the Docker Containerizer or the 
Unified Containerizer.  When setting up your cluster, follow the
appropriate guide based on your requirements.  Note that your Mesos
cluster may be configured to use both the Docker Containerizer and the
Unified Containerizer - in which case follow both sets of installation
instructions.

Calico is particularly suitable for large Mesos deployments on bare 
metal or private clouds, where the performance and complexity costs of 
overlay networks can become significant. It can also be used in public 
clouds.

If you have an existing Mesos cluster, follow the appropriate
installation guide. However, please ensure that Mesos (and Marathon
if you are using it) are installed at the appropriate minimum
version, upgrading if necessary.

## Guides

To build a new Mesos cluster with Calico networking, try one of the
following guides:

#### Quick Start a Sample Cluster:
- [CentOS Vagrant guide](VagrantCentOS.md) will set up a Calico-ready
Mesos cluster, including the Docker containerizer

#### Installation guides:
- [Mesos Cluster Preparation Guide](MesosClusterPreparation.md) provides
details on how to install the required services for a Calico Mesos cluster.
- [Docker Containerizer Usage Guide](UsageGuideDockerContainerizer.md)
explains how to configure and launch tasks with Calico using the
Docker containerizer.
- [Manual Calico Mesos Install Guide](ManualInstallCalicoMesos.md) explains
how to configure a Calico cluster that uses the Mesos unified containerizer.

#### Demonstration guides:
- [Stars demo](stars-demo/) uses the Docker containerizer to show
a network policy visualization of how a Calico cluster is configured.

#### Troubleshooting 
- The [Troubleshooting doc](Troubleshooting.md) contains concise solutions to
issues that users have run into in the past.

[![Analytics](https://calico-ga-beacon.appspot.com/UA-52125893-3/calico-containers/docs/cni/kubernetes/README.md?pixel)](https://github.com/igrigorik/ga-beacon)
