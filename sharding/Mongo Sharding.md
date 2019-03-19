# MONGO SHARDING

![](https://github.com/nguyensjsu/cmpe281-rohank2002/blob/master/Images/Sharding.png)
Create 2 Security Groups:
Internal Access: Open Ports 27017-27019
External Access: Open Ports 27017

### Launch first config server
Configure mongo file
```sh
sudo vi /etc/yum.repos.d/mongodb-org-3.4.repo

[mongodb-org-3.4]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/amazon/2013.03/mongodb-org/3.4/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-3.4.asc
```
Install Mongo
```sh
sudo yum install -y mongodb-org-3.4.10
```
### Start Mongo on restart
```sh
sudo chkconfig mongod on
sudo mkdir -p /data/db
sudo chown -R mongod:mongod /data/db
set config server
sudo mongod --configsvr --replSet crs --dbpath /data/db --port 27019 --logpath /var/log/mongodb/mongod.log --fork
 ```
To check process is running
```sh
ps -aux | grep mongod
sudo kill -9 process id
```

Trying using config file
```sh
sudo vi /etc/mongod.conf
```
change dbpath to /data/db

port to 27019

Comment bind ip

Replication:
replSetName:crs

sharding:

clusterRole:configsvr

start using: ```sh sudo mongod --config /etc/mongod.conf --logpath /var/log/mongodb/mongod.log ```

To check port 
```sh
sudo lsof -iTCP -sTCP:LISTEN | grep mongo
```
Open mongo shell

```sh
mongo -port 27019
```
Initiate replica set
```sh
rs.initiate(
{
	_id:"crs",
	configsvr: true,
	members:[
        {_id : 0, host : "54.214.84.160:27019"},
        {_id : 1, host : "34.221.178.75:27019"}
	]
}
)
rs.status()
```
Go to shard members
```sh
sudo mkdir -p /data/db
sudo chown -R mongod:mongod /data/db
sudo vi /etc/mongod.conf
dbPath:/data/db
port:27018
comment bind ip
replSetName: rs0/rs1
sharding:
clusterRole: shardsvr
```
start using: ```sh sudo mongod --config /etc/mongod.conf --logpath /var/log/mongodb/mongod.log ```

Connect to shard 1 replica 1
```sh
mongo -port 27018
rs.initiate(
{
	_id:"rs0",
	members:[
        {_id : 0, host : "54.191.155.232:27018"},
        {_id : 1, host : "54.244.209.80:27018"}
	]
}
)
```
Shard 2 Replica 2
```sh
rs.initiate(
{
	_id:"rs1",
	members:[
        {_id : 0, host : "34.222.48.104:27018"},
        {_id : 1, host : "54.149.187.225:27018"}
	]
}
)
```
### Mongos(Query Router)
```sh
sudo yum install -y mongodb-org-mongos-3.4.17
sudo vi /etc/mongod.conf
comment storage
db
journal

comment bindip
sharding:
configDB: crs/54.214.84.160:27019,34.221.178.75:27019
start using: sudo mongos --config /etc/mongod.conf --logpath /var/log/mongodb/mongod.log
mongos --configdb crs/54.214.84.160:27019,34.221.178.75:27019
```
Add Shard to mongos
```sh
mongo -port 27017 
sh.addShard("rs0/54.191.155.232:27018,54.244.209.80:27018");
sh.addShard("rs1/34.222.48.104:27018,54.149.187.225:27018");
```
To check shards
```sh
db.adminCommand({listShards:1})
```
Add Data
```sh
use testdb
```
Switch to admin
```sh
use admin
db.runCommand({enablesharding:"testdb"})
db.bios.ensureIndex( { _id : "hashed" } )
sh.shardCollection( "testdb.bios", { "_id" : "hashed" } )
```
Get Shard Distribution
```sh
db.bios.getShardDistribution()
```

Types of sharding:

Hash based sharding:

Based on hased key values, then each chuck is assigned a range

Range sharding:

Collection is divided into ranges[min,max] determined by shard key


sharding bios

```sh
db.runCommand({shardcollection:"testdb.bios",key:{name:1}});
```
[ScreenShots](https://github.com/nguyensjsu/cmpe281-rohank2002/blob/master/sharding/Sharding%20ScreenShots.md)
