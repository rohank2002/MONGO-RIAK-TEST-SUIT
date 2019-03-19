# WEEKLY TIMELINE

# Week 1
Learning about GOAPI,its integration with mongo DB.Establishment of idea replicas in MongoDB.
# Week 2
Mongo DB cluster developed using Ubuntu Servers on Amazon Web Services.
Development of GOAPI and testing with Postman.The PUT method in the cart was difficult to implement,so part code was completed , 
and rest all methods were tested.
# Week 3
Research on RIAK , deployment of cluster and testing partition tolerance (AP) propertires of riak.Completion of RIAK Database Cluster Setup.One of the major issue was creating a partition, even after changing security groups to close ports, the instances still could connect to each other. Many different methods were tried , then for riak IP tables drop was used, which worked. As in the case of mongo, VPC peering network ensured connectivity between regions , so removing it created a partition, hence simulation of partition was taken care of by the above method.
# Week 4
Completed Mongo and RIAK, research on sharding.Documented sharding from various websites, and tried to implement. After the configuration on AWS was done there was some issue as the sharding command wasnt working, shards couldnt get relfected on the sharding databases.So, some more research was required.
# Week 5
The sharding issue was sorted out by reading one of the blog post, issue was I was previously using name field for sharding, Mongo requires some unique field for sharding. There was 50-50 distribution of the data amongst the shards. Video was taken for all the assignments , only thing left is uploading screenshots.
### Week 6
Video was made, screenshots updated for all , answered all test case questions.
