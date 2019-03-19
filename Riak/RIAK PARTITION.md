### RIAK CLUSTER SETUP
1. AMI:             Riak KV 2.2 Series
2. Instance Type:   t2.micro
3. VPC:             cmpe281
4. Network:         private subnet
5. Auto Public IP:  no
6. Security Group:  riak-cluster 
7. SG Open Ports:   (see below)
8. Key Pair:        cmpe281-us-west-1

Riak Cluster Security Group (Open Ports):

    22 (SSH)
    8087 (Riak Protocol Buffers Interface)
    8098 (Riak HTTP Interface)
    Port range: 4369
    Port range: 6000-7999
    Port range: 8099
    Port range: 9080
### Create a JUMPBOX to access this instance
ssh -i <key>.pem ec2-user@<private ip> (access riak node)
### Setup up cluster of 3 nodes
On each node execute:
```sh
sudo riak start
```
For all other nodes, use the internal IP address of the first node:
```sh
sudo riak-admin cluster join riak@<ip.of.first.node>
```
After all of the nodes are joined, execute the following:
```sh
sudo riak-admin cluster plan
sudo riak-admin cluster status
```
If everything looks fine
```sh
sudo riak-admin cluster plan
sudo riak-admin cluster status
```
To check member status
```sh
sudo riak-admin cluster commit
```
check cluster status
```sh
sudo riak-admin member_status
```
### Test
```sh
curl -i http://<Private IP>:8098/buckets?buckets=true
curl -v -XPUT -d '{"foo":"bar"}' \
    http://<Private IP>:8098/buckets/bucket/keys/key1?returnbody=true

curl -i http://<Private IP>:8098/buckets/bucket/keys/key1
```




### Instance IP in Private Network 
```sh
RIAK 1 12.0.1.9
RIAK 2 12.0.1.122
RIAK 3 12.0.1.165
RIAK 4 12.0.1.48
RIAK 5 12.0.1.189
KONG 34.239.180.96
```
### Setup Docker/Kong on JUMPBOX
```sh
sudo yum update -y
sudo yum install -y docker
sudo service docker start
sudo usermod -a -G docker ec2-user


docker network create --driver bridge gumball
docker run -d --name kong-database --network gumball -p 9402:9402 cassandra:2.2
docker run -d --name kong \
         --network gumball \
         -e "KONG_DATABASE=cassandra" \
         -e "KONG_CASSANDRA_CONTACT_POINTS=kong-database" \
         -e "KONG_PG_HOST=kong-database" \
         -p 8000:8000 \
         -p 8443:8443 \
         -p 8001:8001 \
         -p 7946:7946 \
         -p 7946:7946/udp \
         kong:0.9.9
```

### Check Container Status
```sh
docker ps --all --format "table {{.ID}}\t{{.Names}}\t{{.Image}}\t{{.Status}}\t"
```
### Check Container Status
```sh
sudo riak-admin cluster status
```
### Setup RIAK on All Clusters
```sh
sudo riak start (On all Clusters)
sudo riak-admin cluster join riak@12.0.1.9
sudo riak-admin cluster plan
sudo riak-admin cluster status
sudo riak-admin cluster commit
sudo riak-admin member_status
```
### Setup Up Kong To access from/Register on Postman Postman

### Create bucket on node which will be propagated to all(On any one Node)
```sh
sudo riak-admin bucket-type create subjects
sudo riak-admin bucket-type activate subjects
```
### Blocking Traffic to create network partition
Blocking node 1,2,3 on node 4 and 5 :
```sh
sudo iptables -I INPUT -s 12.0.1.9 -j DROP
sudo iptables -I INPUT -s 12.0.1.122 -j DROP
sudo iptables -I INPUT -s 12.0.1.165 -j DROP
sudo iptables -I INPUT -s 12.0.1.48 -j DROP
sudo iptables -I INPUT -s 12.0.1.189 -j DROP
```
Unblock Traffic :
```sh
sudo iptables -D  INPUT -s 12.0.1.9 -j DROP
sudo iptables -D INPUT -s 12.0.1.122 -j DROP
sudo iptables -D INPUT -s 12.0.1.165 -j DROP
sudo iptables -D INPUT -s 12.0.1.48 -j DROP
sudo iptables -D INPUT -s 12.0.1.189 -j DROP
```
### Test Using Postman
### [Sceenshots](https://github.com/nguyensjsu/cmpe281-rohank2002/blob/master/Riak/RIAK%20Personal%20Project.pdf)

### [TimeLine](https://github.com/nguyensjsu/cmpe281-rohank2002/blob/master/Timeline.md)
 
### Test Case
| Sr No. | Test Case | Result |
|-------|-------|-------|
| 1. | Checking  of data before partition on each node | All the nodes showed consistent data|
| 2. | Checking Consistency of data before partition after insertion | All the nodes showed consistent data|
| 3. | Checking Consistency of data after partition | Stale data in the nodes which are cut-off from the ring, so stale data is fetched though availability is achieved|
| 4. | Adding data to one of the node in the ring and then one which is cutoff | Inconsistent Data in nodes |
| 5. |Scenario after recovery | Last written value has higher timestamp so , propagated to all nodes. |
| 6. | Switch off any of the node | No effect on the ring, configuration works very well.|

### Test Questions:
AP: 
How does the system function during normal mode (i.e. no partition)?

Absence of master , leads to read and write to any node in the cluster(Ring).The nodes communicate with each other to keep themselves updated.

What happens to the nodes during a partition? 

The nodes which were partitioned from the ring, were also accesible. Data added to the ring is not visible on the partitioned node, and data added to the partitioned node is not available to the nodes in the ring.

Can stale data be read from a node during a partition? 

Yes, stale data could be read after the partition on the partitioned nodes.

What happens to the system during partition recovery?

As mentioned in 2nd question's answer, data was added to both the sides i.e. partioned and the ring, on recovery from partition, data which was latest was propagated to all nodes, and same in demonstrated in the video.

### Challenges Faced

1. Tried docker container provided by riak to complete the assignment, was more complicated so continued with AWS
2. Security Groups did not work so with some internet surfing figured out that ip tables drop, can simulate partition.
