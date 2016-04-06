<!--- master only -->
> ![warning](images/warning.png) This document applies to the HEAD of the calico-mesos-deployments source tree.
>
> View the calico-mesos-deployments documentation for the latest release [here](https://github.com/projectcalico/calico-mesos-deployments/blob/0.26.0%2B1/README.md).
<!--- else
> You are viewing the calico-mesos-deployments documentation for release **release**.
<!--- end of master only -->

# Using Calico Mesos and the Unified Containerizer
The following information includes application json and information on launching tasks in a Mesos Cluster with Calico.

## Launching Tasks with Marathon
Calico is compatible with all frameworks which use the new NetworkInfo protobuf when
launching tasks. Marathon has introduced limited support for this in v0.14.0.

### Launching Marathon
If you have not installed Marathon, you can run Marathon directly by running
the commands [here](MesosClusterPreparation.md#marathon), or you can quickly
start it as a Docker container like the following:

```
docker run \
-e MARATHON_MASTER=zk://<ZOOKEEPER_IP>:2181/mesos \
-e MARATHON_ZK=zk://<ZOOKEEPER_IP>:2181/marathon \
-p 8080:8080 \
mesosphere/marathon:v0.14.0
```

### Launching Tasks
Marathon-v0.14.0 supports two new fields in an application's JSON file:

- `ipAddress`: Specifiying this field grants the application an IP Address
networked by Calico.
- `groups`: Groups are roughly equivalent to Calico Profiles. The default
implementation isolates applications so they can only communicate with
other applications in the same group. Assign the static `public` group
to a task to allow it to communicate with any other application.
 
> See [Marathon's IP-Per-Task documentation][marathon-ip-per-task-doc] for more information.

The Marathon UI does not yet include a field for specifiying NetworkInfo,
so we'll use the command line to launch an app with Marathon's REST API.
Below is a sample `app.json` file that is configured to receive an address
from Calico:

```
{
    "id": "hello-world-1",
    "cmd": "ip addr && sleep 30",
    "cpus": 0.1,
    "mem": 64.0,
    "ipAddress": {
        "groups": ["my-group-1"]
}
```

Send the `app.json` to Marathon to launch it:

	curl -X POST -H "Content-Type: application/json" http://localhost:8080/v2/apps -d @app.json

#### Launching Docker Images
The release of Mesos 0.27.0 includes changes to the Mesos Containerizer which enable it to launch docker images. Using an experimental build of Marathon (`djosborne/marathon:docker`), we can launch  docker images networked with calico with the following json blob:

```
{
    "id": "unified-1",
    "cmd": "ip addr && sleep 30",
    "cpus": 0.1,
    "mem": 64.0,
    "ipAddress": {
        "groups": ["my-group-1"]
    },
    "container": {
        "type": "MESOS",
        "mesos": {
            "image": {
                "type": "DOCKER",
                "docker": {
                    "name": "ubuntu:14.04"
                }
            }
        }
    }
}
```

[calico-slack]: https://calicousers-slackin.herokuapp.com/
[marathon-ip-per-task-doc]: https://github.com/mesosphere/marathon/blob/v0.14.0/docs/docs/ip-per-task.md
[![Analytics](https://calico-ga-beacon.appspot.com/UA-52125893-3/calico-containers/docs/mesos/README.md?pixel)](https://github.com/igrigorik/ga-beacon)
