# cmpe281-rohank2002
 
Understanding the CAP theorem by implementation of Mongo CP and RIAK AP databases .A Go API is used to hit these databases to produce modifications.

### Test Questions:

CP:

How does the system function during normal mode (i.e. no partition)
The Master Node controls all the slaves, any data which is written hits the master and master sends it to respective slave.

What happens to the master node during a partition?
The master node detects that the communication is not possible with the slaves(Heartbeat) which are not available, it updates data on the nodes that are available. 

Can stale data be read from a slave node during a partition?
If we are reading data from a slave which is not partitioned, new data is read.But if we execute a mongo command on the slave of the partitioned region stale data is present.

What happens to the system during partition recovery?
After Partition recovery , the master detects the slaves are available now, so it updates the data on them.No stale data can be found in any node after partition.

AP:
How does the system function during normal mode (i.e. no partition)
Absence of master , leads to read and write to any node in the cluster(Ring).The nodes communicate with each other to keep themselves updated.

What happens to the nodes during a partition? 
The nodes which were partitioned from the ring, were also accesible. Data added to the ring is not visible on the partitioned node, and data added to the partitioned node is not available to the nodes in the ring.

Can stale data be read from a node during a partition?
Yes, stale data could be read after the partition on the partitioned nodes.

What happens to the system during partition recovery?
As mentioned in 2nd question's answer, data was added to both the sides i.e. partioned and the ring, on recovery from partition, data which was latest was propagated to all nodes, and same in demonstrated in the video.

[Weekly TimeLine](https://github.com/nguyensjsu/cmpe281-rohank2002/blob/master/Timeline.md)

### Documentation

[Mongo Documentation](https://github.com/nguyensjsu/cmpe281-rohank2002/blob/master/Mongo/Mongo%20Cluster%20setup.md)

[RIAK Cluster Setup Documentation](https://github.com/nguyensjsu/cmpe281-rohank2002/blob/master/Riak/RIAK%20Cluster.md)

[RIAK Partitioning Documentation](https://github.com/nguyensjsu/cmpe281-rohank2002/blob/master/Riak/RIAK%20PARTITION.md)

[MongoDB Sharding](https://github.com/nguyensjsu/cmpe281-rohank2002/blob/master/sharding/Mongo%20Sharding.md)

### Screenshots

[RIAK screenshot](https://github.com/nguyensjsu/cmpe281-rohank2002/blob/master/Riak/RIAK%20Personal%20Project.pdf)

[MONGO screenshots](https://github.com/nguyensjsu/cmpe281-rohank2002/blob/master/Mongo/Mongo%20ScreenShots.md)

[Sharding screenshots](https://github.com/nguyensjsu/cmpe281-rohank2002/blob/master/sharding/Sharding%20ScreenShots.md)

 
