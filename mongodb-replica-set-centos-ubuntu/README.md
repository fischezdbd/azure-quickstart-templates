# Install MongoDB Replica Set


[![Deploy To Azure](https://raw.githubusercontent.com/fischezdbd/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.svg?sanitize=true)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Ffischezdbd%2Fazure-quickstart-templates%2Fmaster%2Fmongodb-replica-set-centos-ubuntu%2Fazuredeploy.json)
[![Visualize](https://raw.githubusercontent.com/fischezdbd/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/visualizebutton.svg?sanitize=true)](http://armviz.io/#/?load=https%3A%2F%2Fraw.githubusercontent.com%2Ffischezdbd%2Fazure-quickstart-templates%2Fmaster%2Fmongodb-replica-set-centos-ubuntu%2Fazuredeploy.json)

This template deploys a MongoDB Replica Set on CentOS and enables Zabbix monitoring, and allows user to define the number of secondary nodes. The replica set has a primary node, 2 secondary nodes by default.

This template also allows you to input your existing zabbix server IP address to monitor these MongoDB nodes.

The replica set nodes are exposed on public IP addresses that you can access through SSH on the standard port, also mongodb port 27017 open.

The nodes are under the same subnet 10.0.1.0/24. The primary node ip is 10.0.1.240, the secondary nodes ip address start from 10.0.1.4. For example:

- primary node ip: 10.0.1.240

- secondary node 1 ip: 10.0.1.4

- secondary node 2 ip: 10.0.1.5

## Important Notice
Each VM of the replica set uses raid0 to improve performance. We use 4 data disks on each VM for raid0. The size of data disks(setup raid0) on each VM are determined by yourself. However, there is size of data disks limit per the VM size. Before you set the size of data disks, please refer to the link https://azure.microsoft.com/en-us/documentation/articles/virtual-machines-linux-sizes/ for the correct choice.

## After deployment, you can do below to verify if the replica set really works or not:

1. SSH connect to primary node, execute below
  ```
  $mongo -u "<mongouser>" -p "<mongopassword>" "admin"

  rs.status()

  exit
  ```

  Upper rs.status() command will show the replica set details. 

2. You can also check the data replication status. SSH connect to primary node, execute below:
  ```
  $mongo -u "<mongouser>" -p "<mongopassword>" "admin"

  use test

  db.mycol.insert({"title":"MongoDB Overview"})

  db.mycol.find()
  ```

- 2.1 SSH connect to secondary nodes, execute below
  ```
  $mongo -u "<mongouser>" -p "<mongopassword>" "admin"

  use test

  db.getMongo().setSlaveOk()

  show collections

  db.mycol.find()
  ```

- 2.2 If db.mycol.find() command can show the result like primary node does, then means the replica set works.

## Known Limitations
- The MongoDB version is limited to 4.4, 4.2, 4.0, 3.8, 3.6. Some versions of MongoDB are only configured to bind to localhost (and therefore will not allow connections from the internet). To change this, see the /etc/mongod.conf for binding ip's.
- We expose all the nodes on public addresses so that you can access MongoDB service through internet directly.
- MongoDB suggests that the replica set has an odd number of voting members. So the number of secondary nodes is better to set to even number, like 2, 4 or 6, then plus the primary node, fill the requirement that the replica set has an odd number of voting members.
- A replica set can have up to 50 members, but only 7 voting members. So the maximum number of secondary nodes is 6.
- The replica set doesn't have arbiter nodes.
- The replica set enables internal authentication. Check /etc/mongokeyfile for details.
- More MongoDB usage details please visit MongoDB website https://www.mongodb.org/ .

## More info on Azure VM sizes for MongoDB
1.  A  series

A series offers general purpose instances that fit most workloads. They are available in various sizes ranging from 0.75 GB to 56 GB. 
Inside A series you are offered two options – ‘Basic’ and ‘Standard’.  
The ‘Basic’ version costs less but does not offer load balancing, auto-scaling etc. 
From a database perspective, the most important difference is that with ‘Basic’ instances your azures disks (page blobs) are limited to 300 IOPS/disk whereas with ‘Standard’ instances you can go up to 500 IOPS/disk. 
This can make a big difference, especially with larger instances when you can RAID the disks. 
Our recommendation is to use ‘Standard’ machines whenever possible to leverage the enhanced I/O. 
The number of disks that can be attached to a VM depends on the size of the VM. 
You can go up to 16 disks for  A7 machine. More details can be found here: https://docs.microsoft.com/en-us/azure/cloud-services/cloud-services-sizes-specs

2.  D series/ DS series

D series instances offer better performance compared to the A series  - specifically better CPU and local SSD instances. 
The local SSD disk will give you the best disk performance possible on Azure. 
However, it is called ‘local’ for a reason. 
The data on these disks is ephemeral – if for any reason your VM is stopped you will lose all the data on your disk. 
So the Local SSD should not be used as a primary store. 
The DS series is more interesting from a data perspective because it is the only instance type that supports Premium storage. 
Premium storage as the name suggests offers enhanced disk IOPS depending on the size of the disk. 
If possible try to use premium storage for all your data disks. For more details consult the Premium storage overview.

Disk Types	P10	P20	P30
Disk Size	128 GB	512 GB	1024 GB
IOPS per Disk	500	2300	5000
Throughput per Disk	100 MB/sec	150 MB/sec	200 MB/sec

3.  G series

This is the ‘monster’ series offering huge amounts of RAM (up to 448 GB) and local SSD. 
If you can afford it this series offers the best performance.  
The G series instances might only be available in the West US and East US 2 datacenters.
