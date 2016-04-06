<!--- master only -->
> ![warning](../../images/warning.png) This document applies to the HEAD of the calico-docker source tree.
>
> View the calico-docker documentation for the latest release [here](https://github.com/projectcalico/calico-containers/blob/v0.17.0/README.md).
<!--- else
> You are viewing the calico-docker documentation for release **release**.
<!--- end of master only -->

# Troubleshooting Calico with Mesos
This article contains Mesos specific troubleshooting advice for Calico.  See also the [main Calico troubleshooting](../../Troubleshooting.md) guide.

## Test Cluster Health
Before you start launching tasks with Marathon, we suggest running the
Calico-Mesos Test Framework. This Framework will register with mesos directly
and launch ping and sleep tasks across your mesos cluster, verifiying netgroup
enforcement and network connectivity between tasks.

To launch the framework, run the following docker command from any host that can communicate with your master (we recommend simply running it directly on the master itself):

	docker run --net=host calico/calico-mesos-framework <MASTER_IP>:5050

- Some tests require multiple hosts to ensure cross-host communication, and
may fail unless you are running 2+ agents.
- Additionally, if running your cluster in the public cloud, cross-host tests
will fail unless you [Enable IP over IP]
(https://github.com/projectcalico/calico-containers/blob/master/docs/FAQ.md#can-i-run-calico-in-a-public-cloud-environment)

Be sure to contact us on [Slack][calico-slack] if your tests are still not passing!


[![Analytics](https://calico-ga-beacon.appspot.com/UA-52125893-3/calico-containers/docs/cni/kubernetes/Troubleshooting.md?pixel)](https://github.com/igrigorik/ga-beacon)
