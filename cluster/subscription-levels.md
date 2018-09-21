---
lastmod: 2017-01-21
date: 2017-01-21
title: Cluster Subscription
linktitle: Subscription levels
notoc: true
menu:
  documentation:
    parent: cluster
weight: 10
---
<div class="alert alert-warning alert-dismissible">
    <button type="button" class="close" data-dismiss="alert" aria-hidden="true">×</button>
    <p><i class="icon fa fa-warning"></i> Alert!</p>
    The cluster version is not available in the open source distribution.
</div>

When you start the Cluster Manager the binary will validate against the server the subscription level. It will behave
according to your subscription. For instance, if your subscription allows to have 10 nodes in the cluster it won't allow
 to add the 11th node unless you upgrade. The cluster manager will check daily the subscription.

 The subscription validation is very tolerant. This means that if for instance when the subscription is validated there
 is trouble reaching the server the cluster manager can keep trying for several days without affecting your service
 conditions. Even when the cluster manager detects that your subscription has expired you will have some days margin
 to renew it before the nodes are degraded to the "free" version.
