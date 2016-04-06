<!--- master only -->
> ![warning](images/warning.png) This document applies to the HEAD of the calico-mesos-deployments source tree.
>
> View the calico-mesos-deployments documentation for the latest release [here](https://github.com/projectcalico/calico-mesos-deployments/blob/0.27.0%2B2/README.md).
<!--- else
> You are viewing the calico-mesos-deployments documentation for release **release**.
<!--- end of master only -->

# Installing mesos-master, zookeeper, etcd, and marathon
This guide details a minimal install of the external services required by a Calico-Mesos Agent.
- **A Note on Clustered Services**: In general, a production-ready mesos cluster should run each of these services across multiple highly available hosts with a quorum. However, for demo & development purposes, **this guide will cover launching only one of each service on a single centos7 host.** Consult the appropriate documentation for each service:
    - [etcd clustering](https://coreos.com/etcd/docs/latest/clustering.html)
    - [zookeeper multi-server setup](https://zookeeper.apache.org/doc/r3.3.2/zookeeperAdmin.html#sc_zkMulitServerSetup)
    - [mesos high-availability mode](http://mesos.apache.org/documentation/latest/high-availability/)

    For more information on how to set up a more production-ready calico-mesos cluster in such a manner, [contact us on slack][slack].
- **A Note on Calico's Customizations in Mesos**: It is important to note that no customizations are necessary for any of the aformentioned services to be compatible with a Calico-Mesos Agent - **Adding calico to a mesos cluster only requires modifications to each Agent.**


## Prerequisite: Add the Mesos Official Repository
Mesos-master, zookeeper, and marathon are installed from the official Mesos repository. Add the official repository:

    rpm -Uvh http://repos.mesosphere.com/el/7/noarch/RPMS/mesosphere-el-repo-7-1.noarch.rpm

## ZooKeeper
Install Mesos' datastore - ZooKeeper:

Manual install:

```
yum install -y zookeeper
systemctl start zookeeper
```

Docker install:

```
sudo docker pull jplock/zookeeper:3.4.5
sudo docker run --detach --name zookeeper -p 2181:2181 jplock/zookeeper:3.4.5
```

#### Permit Zookeeper in firewall
ZooKeeper uses tcp over port 2181. If you're using a firewall, open this port:

```
sudo firewall-cmd --zone=public --add-port=2181/tcp --permanent
sudo systemctl restart firewalld
```

## Mesos Master
Supported Versions: `0.28` (recommended, required for Docker Containerizer),
`v0.27`, `v0.26.0` (deprecated)

With Zookeeper up and running, we can launch our mesos-master process. Run the following commands, replacing `$IP` with the cluster-accessible IP you would like Master to bind to.

Manual install: 

```
yum install -y mesos
echo $IP > /etc/mesos-master/hostname
echo $IP > /etc/mesos-master/ip
systemctl start mesos-master
```

Docker install:


## Marathon
Supported Versions: `v1.0.0` (recommended), `v0.14.0-v0.14.2`

Once mesos-master is running, we can launch Marathon:

```
sudo yum install -y marathon
systemctl start marathon
```

Marathon listens for tcp connections on port 8080. Open this port on your firewall:

```
sudo firewall-cmd --zone=public --add-port=8080/tcp --permanent
sudo systemctl restart firewalld
```

#### Marathon as Docker service


## etcd
Calico uses etcd as its data store and communication mechanism among Calico components. Install it from Centos' repositories with the following command, replacing `$IP` with the cluster-accessible IP you would like etcd to bind to.

Manual install:

```
yum install -y etcd
echo ETCD_LISTEN_CLIENT_URLS=\"http://0.0.0.0:2379\" >> /etc/etcd/etcd.conf
echo ETCD_ADVERTISE_CLIENT_URLS=\"http://$IP:2379\" >> /etc/etcd/etcd.conf
systemctl start etcd.service
```

Docker install:

```
sudo docker pull quay.io/coreos/etcd:v2.2.0
sudo mkdir -p /var/etcd
sudo docker run --detach --name etcd --net host -v /var/etcd:/data quay.io/coreos/etcd:v2.2.0 \
     --advertise-client-urls "http://$IP:2379,http://${FQDN}:4001" \
     --listen-client-urls "http://0.0.0.0:2379,http://0.0.0.0:4001" \
     --data-dir /data
```

#### Permit etcd in firewall
Etcd listens for tcp connections on ports 2379. Open these ports on your firewall:

```
sudo firewall-cmd --zone=public --add-port=2379/tcp --permanent
sudo systemctl restart firewalld
```

#### etcd as Docker service

## Next steps
See [Our Guide on Using Calico-Mesos](UsingCalicoMesos.md) for info on how to test your cluster and start launching tasks networked with Calico.

[slack]: https://calicousers.slack.com
[![Analytics](https://calico-ga-beacon.appspot.com/UA-52125893-3/calico-containers/docs/mesos/MesosClusterPreparation.md?pixel)](https://github.com/igrigorik/ga-beacon)

