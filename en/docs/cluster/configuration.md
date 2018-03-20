---
lastmod: 2017-01-21
date: 2017-01-21
linktitle: Cluster configuration
menu:
  main:
    parent: cluster
title: Cluster configuration
weight: 10
notoc: true
---
<div class="alert alert-warning alert-dismissible">
    <button type="button" class="close" data-dismiss="alert" aria-hidden="true">×</button>
    <p><i class="icon fa fa-warning"></i> Alert!</p>
    The cluster version is not available in the open source distribution.
</div>

Running a cluster basically requires the conditions:

- The Cluster Manager is running
- The KrakenD nodes configurations has the cluster mode enabled

# Starting the KrakenD Cluster Manager
After downloading the cluster manager and installing it you can start it using:

    krakend-cluster-manager

You can have several cluster managers running as long as they conform a unique group. This will give you high availability
for the cluster manager itself.


# Adding nodes to the cluster
In order to add nodes to the cluster, make sure to enable the clustering mode in its service configuration. To do so edit
 the file `/etc/krakend/service.yml` and put the DNS or the IP of the machine running the Cluster Manager. For instance:

    seed: 192.168.1.100:7000

Then start the KrakenD service with no special parameter. Once it has started you will see in the log of the cluster a
message like this:

    CM  2017/02/13 - 18:18:23.290 ▶ INFO Sending the limits to the krakend: krakend_3f20a44d-3d93-4bc9-829b-b76b22055142

The message shows the ID that has been assigned to the new node (`3f20a44d-3d93-4bc9-829b-b76b22055142` in this example).

If your subscription has reached the maximum number of nodes you'll see a message like this in the cluster manager log:

    Your current subscription does not allow more nodes running. Please upgrade to add more nodes.

You can always upgrade your subscription and add more nodes.

# Removing nodes from the cluster
Removing a node from the cluster only requires to stop or kill the KrakenD service in the desired node. After stopping
it the Cluster Manager will immediately mark the node as removed, but the statistics will last while the time window is valid.

If for any reason a node leaves unexpectedly the cluster, like in a network failure, the node is marked as unreachable and if
 after a grace period there is no communication saying otherwise it will be marked as down. Any node that has left can
 rejoin the cluster anytime.
