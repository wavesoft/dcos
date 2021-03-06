## etcd backup and restore

This section describes the process of backing up and restoring the state of etcd running inside a DC/OS cluster.

The etcd cluster is possible to recover from temporary failures and tolerates up to (N-1)/2 permanent failures for a cluster with N members. However, the following scenarios, the operator is recommended to backup and restore etcd data:
- to recover from a disastrous failure to return a cluster to a healthy state
- to perform maintenance operations, such as an upgrade or downgrade

As only etcd v3 keys require to be stored persistently in DC/OS, only v3 keys are backed up through snapshot and restored in the following process.

NOTE: As etcd relies on ZooKeeper in DC/OS, the operator MUST maintain etcd while ZooKeepr functions normally.

## Backing up an etcd cluster

etcd cluster achieves reliable distributed coordination and hence key/value data consistency among all etcd instances, etcd instances on all masters are capable of providing all the key/value data in the current state. As we expose the etcd instance from the Mesos leader node outside to the internal and external clients, it's reasonable to backup the etcd data on the leader node of the DC/OS cluster.

#### Prerequisites

- Make sure there is enough disk space available to temporarily store the etcd backup on a particular master node.
- Any shell commands must be issued as a privileged Linux user.

1. log into the leader node and backup etcd via the following provided script.

`/opt/mesosphere/bin/dcos-shell dcos-etcdctl backup <backup-tar-archive-path>`

2. Download the created etcd backup tar archive from this leader node to a safe location outside of the DC/OS cluster.

3. Remove the etcd backup tar archive from the leader node.

## Restoring an etcd cluster 

The operator is supposed to execute all the following steps on all master nodes.

1. Copy the previously created single etcd backup tar archive to every master node’s file system.

2. Stop the etcd instances on all master nodes via the systemd unit.

`systemctl stop dcos-etcd`

3. Initiate the restore procedure via the provided DC/OS etcd restore script.

`/opt/mesosphere/bin/dcos-shell dcos-etcdctl restore <backup-tar-archive-path>`

4. Start the previously stopped etcd instances.

`systemctl start dcos-etcd`

5. Check the etcd cluster status.

Until now, the etcd cluster is expected to be recovered from the backup file.

`/opt/mesosphere/bin/dcos-shell dcos-etcdctl diagnostic`

The above command typically presents the results of etcdctl command [`endpoint health`](https://github.com/etcd-io/etcd/tree/master/etcdctl#endpoint-health) and [`member list -w json`](https://github.com/etcd-io/etcd/tree/master/etcdctl#member-list) for now, and a healthy etcd cluster should meet the following requirement given the output of the above commands on all master nodes:

- `endpoint health` checks the healthiness of on the current etcd instance, which should meet `healthy` 
- `member list -w json` returns the cluster members, which should return all etcd instances
