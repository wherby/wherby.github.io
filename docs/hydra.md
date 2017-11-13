#Using Akka Cluster to create HA container
      --- Introduction to Hydra
Suppose there have a distributed application running on many nodes(VMs or hosts), and the application may crash on some nodes, or some nodes may fail silently. 

How can you detect the node failure or application crash easily? If you setup a normal health check task for every hosts, the check time is linear time O(n) to the node number and the check task or the node (for the check task) both also may be crashed.

The Akka Cluster use very efficient way to detect node failure[https://doc.akka.io/docs/akka/current/scala/common/cluster.html ], and if on each node, the application status can be verified, then the both check will be done in consitant time O(C).

The Hydra Cluster is based on Akka Cluster, which is use a container to deploy application:
If the application crash, the container will report to Hydra, Hydra will redeploy the application to another node.
If the node fail, then the Akka Cluster will detect the node failure and then report the failure to Hydra, then Hydra will redeploy the apps on the node to other nodes.

For more code see: https://github.com/wherby/Hydra
For test of Hydra see: https://github.com/wherby/HydraRelease/tree/master/0.1.0
