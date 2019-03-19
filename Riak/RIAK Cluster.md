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



