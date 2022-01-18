# File, Block and Object storage

* File: The file type os storage stores all elements in the disk as files, even directories, each file has an address, size.

The process of storing and retrieving files follows a predefined schema `c:\documents... for windows`.

If we want to access a given file through the network, we must put it in a shared directory, this type is called NAS (Network Attached Storage).

This is fine if we have small files and small network, but if we have a huge amount of data and large networks, this will flood our network.

* Block: Block storage is composed of two main units, disks where the data will be stored and which are devided into multiple blocks, and the second component is server area network, which is the logical unit that stores and retrieves data, so the os of the user is no longer responsible of storing and retrieving data, servers are powerfull and uses more advanced login to store and retrieve data.

So the os of the user only makes requests to the servers area network and receive the data. 

This type of storage is fast.

The only stored metadata about files is file address

* Object: This type of storage stores files in a flast structure, and adds a lot of metadata about stored files, this can be used in analytics.

## Ceph

Ceph is a distributed, auto healing, scalable and performant storage solution.

It devides all files into block, each block is replicated into a defined number of replicas, if we lose a disk, ceph, automatically detects that our disk is no longer present, and creates new replicas for our blocks to reach the wanted number.

Each file is devided into blocks, and then stored in a pool of disks, this pool is composed of multiple PG (Placement groups), each PG is placed in a specific disk (OSD).

To access files stored in ceph, we can use block, filesystem or object mode.