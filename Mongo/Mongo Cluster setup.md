# Mongo CLuster Setup for Cart API

![](https://github.com/nguyensjsu/cmpe281-rohank2002/blob/master/Images/mongo%20setup.png)

Create JUMPBOX in both regions, although one can access both but , on deletion on Peer connection, it cannot access other region.




### Launch Instance
1. AMI:             Ubuntu Server 16.04 LTS (HVM)
2. Instance Type:   t2.micro
3. VPC:             cmpe281
4. Network:         private subnet
5. Auto Public IP:  no
6. Security Group:  mongodb-cluster,default
7. SG Open Ports:   22, 27017
8. Key Pair:        cmpe281-us-west-1




### Configure Key
```sh
openssl rand -base64 741 > keyFile
sudo mkdir -p /opt/mongodb
sudo cp keyFile /opt/mongodb
sudo chown mongodb:mongodb /opt/mongodb/keyFile
sudo chmod 0600 /opt/mongodb/keyFile
```
### Configure Mongo Cluster
```sh
sudo vi /etc/mongod.conf
```
1.  remove or comment out bindIp: 127.0.0.1
replace with bindIp: 0.0.0.0 (binds on all ips)
network interfaces
net:
    port: 27017
    bindIp: 0.0.0.0

2. Uncomment security section & add key file

security:
    keyFile: /opt/mongodb/keyFile

3. Uncomment Replication section. Name Replica Set = cmpe281

replication:
   replSetName: cmpe281     

4. Create mongod.service
```sh
sudo vi /etc/systemd/system/mongod.service
```
    [Unit]
        Description=High-performance, schema-free document-oriented database
        After=network.target

    [Service]
        User=mongodb
        ExecStart=/usr/bin/mongod --quiet --config /etc/mongod.conf

    [Install]
        WantedBy=multi-user.target
```sh
sudo systemctl enable mongod.service
sudo service mongod restart
sudo service mongod status
```
### Create AMI
Launch Similar instance , copy AMI into Virginia and launch into private subnet.
### VPC PEERING CONNECTION ON BOTH REGIONS
VPC>>Peer Connections

     Select Requester as current
     
     Select Virginia region
     
     Add acceptor VPC as Virginia VPC ID
     
     Create CONNECTION
     
### Edit/Create Route tables on both REGIONS

Local Default(Cannot be changed)

Destination:(CIDR block of Virginia(12.0.0.0/16))

Target Peer: connection

Destination: (0.0.0.0/0)

Target: (Nat Instance)

### On Primary Instance
```sh
mongo
rs.initiate( {
       _id : "cmpe281",
       members: [
          { _id: 0, host: "11.0.1.171:27017" },
          { _id: 1, host: "11.0.1.96:27017" },
          { _id: 3, host: "11.0.1.193:27017" },
          { _id: 4, host: "12.0.1.209:27017" },
          { _id: 5, host: "12.0.1.140:27017" },
       ]
    })
      rs.status()
```
### Create Admin Account
```sh
use admin
db.createUser( {
       user: "admin",
       pwd: "cmpe281",
       roles: [{ role: "root", db: "admin" }]
   });
```
### Create Docker container in a private instance
Security Group:

Cart: 5000

Default

```sh
sudo yum update -y
sudo yum install -y docker
sudo service docker start
sudo usermod -a -G docker ec2-user
docker run --name cart -p 3000:3000 -td rohank2002/mongocart
```
### Setup Kong on Jumpbox
Security Group:
Kong 8000,8001
Jumpbox 22

```sh
sudo yum install -y docker
sudo service docker start
sudo usermod -a -G docker ec2-user

docker network create --driver bridge kong
docker run -d --name kong-database -p 9402:9402 cassandra:2.2
docker run -d --name kong \
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
Note:
On accessing any slave
```sh
rs.slaveOk()
```
### [Screenshots](https://github.com/nguyensjsu/cmpe281-rohank2002/blob/master/Mongo/Mongo%20ScreenShots.md)
### [Timeline](https://github.com/nguyensjsu/cmpe281-rohank2002/blob/master/Timeline.md)

### Test Case
| Sr No. | Test Case | Result |
|-------|-------|-------|
| 1. | Checking Consistency of data before partition | All the nodes showed consistent data|
| 2. | Checking Consistency of data before partition after insertion | All the nodes showed consistent data|
| 3. | Checking Consistency of data after partition | Stale data on cut-off nodes, so as primary preffered set in Go API, no stale data is fetched|
| 4. | Checking Consistency of data after reccovery from partition | All the nodes showed consistent data, propagated by master|
| 5. | Switch off master | New Master elected, so automatically Go API calls routed to new master|

### Test Questions
CP:

How does the system function during normal mode (i.e. no partition) 

The Master Node controls all the slaves, any data which is written hits the master and master sends it to respective slave.

What happens to the master node during a partition?

The master node detects that the communication is not possible with the slaves(Heartbeat) which are not available, it updates data on the nodes that are available.

Can stale data be read from a slave node during a partition?

If we are reading data from a slave which is not partitioned, new data is read.But if we execute a mongo command on the slave of the partitioned region stale data is present.

What happens to the system during partition recovery?

After Partition recovery , the master detects the slaves are available now, so it updates the data on them.No stale data can be found in any node after partition.

### Challenges Faced 
1. To simulate partition tried editing security groups , didnt work! Changed route table association of private subnet which did the job.
2. The mongo image in one region had to be propagated to another, as all in a cluster should have a same keyfile, used AMI copy service by AWS.
