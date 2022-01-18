# CEPH

## Main components:

### OSD

* Object storage Devices 
* A disk + CPU/RAM/Network (smart component)
* Wa can have thousands of disks
* Storage backend: ceph used to use XFS as a filesystem
* Ceph is NOT a filesystem, it's an algorithm that distribute data across disks.
* Ceph now uses Blustore as a filesystem and not XFS anymore

### PG

* Placement Group
* From 128 to 1024 PG by Pool
* Recommended redundancy is 3 
* A PG is distributed across 3 OSDs, and one of them is primary

### POOLS

* Can have multiple pools in a single cluster
* 1 pool per usage
* Pool SSD, Pool SATA ...
* Possible modes: replication and erasure coding.

### How to access

* Block: RBD
* Filesystem: CephFS
* Object: HTTP REST (Amazon S3 compatible)

### RADOS

* Group of algorithms that allows the client to access the cluster.

### CRUSH

* Controlled Replication Under Scalable Hashing
* Algorithm used to find objects 
* Uses a CURSH map
* Objects are grouped by PG

### How it works

* EACH disk has its own daemon (A machine with 4 disks will have 4 daemons)
* MON: A daemon monitor (1 per machine)
* MGR: A daemon manager (1 per cluster)
* MDS: Metadata daemon (for CephFS) (Optional)
* RGW: Daemon for HTTP REST access (Optional)

## Logical components

### RADOS

* A reliable, autonomous, distributed object store comprised of self-healing, self-managing, intelligent storage nodes. 
* And this is the basic component which is composed of OSDs
* OSD is a logical process that manages a single disk inside the cluster.
* OSDs serve the stored objects to clients 
* OSDs are peered to perform replication and recovery tasks
* RADOS has another components which is the monitor(s)
* A cluster can have as many  monitors as we want but it's recommended to have a small and odd number
* Monitors maintain cluster membership and state and provide consensus for distributed descision making.
* Monitors do not serve objects to clients.

### LIBRADOS

* A library allowing apps to directly access RADOS, with support for java, php, python, c & c++
* We add this library to our app to talk to the ceph cluster, and it uses a socket to communicate.

### RADOSGW

* A bucket-based REST gateway, compatible with s3 and swift.
* RADOSGW sits in between our app and librados, its job is to link an app that speaks REST with the cluster that has its own protocol.

### RBD

* A reliable and fully distributed block device, with a Linux kernel client and a QEMU/KVM driver 
* Collects a buck of chunks of data and present it as a disk for a VM.
* Virtualization integrates with LIBRBD to interact with LIBRADOS.
* And we can use a Linux kernel module called `KRBD` tomap an RBD volume to a linux device.
* The disk is found under /dev, and can be manipulated as any other disk, but it's actually distributed across the cluster.
* Storage of disk images in RADOS
* Decouples VM from Host
* Snapshots
* Copy-on-write clones.
* Supported in (linux, Qemu/Kvm, Openstack, Cloudstack...)

### Ceph FS

* Used to mount filesystems into one or multiple targets.
* It uses POSIX
* Metdata of each filesystem is stored in metadata server (Third components RADOS)
* Clients retrieve only metadata (Permissions, last time of modifs ... ) of filesystems from metadata servers, the data itself is retrieved via OSDs.

### Metadata sevrver

* Manages metadata for POSIX-compliant shared filesystem (Directory hirearchy, file owners, mode ...)
* Stores metadata in RADOS
* Does NOT serve file data to clients.
* Only required for shared filesystems.

## How does clients retrieve data ?

* Clients need to know which OSDs to connect to, to get the data.
* Ceph server and clients Uses the `CRUSH` algorithm to calculate where the data is located, and connect to the appropriate OSD.
* CRUSH: Controlled Replication Under Scalable Hashing.
* When the client wants to store a buch of bits in the cluster, it hashes the object name and devide it by the num of placement groups, to define which PG will contain the data.
* Then, it calls the `CRUSH` algorithm and pass the (pg, cluster state, rule set) to it, and based on that, the algorithm will calculate where the data will be stored in the cluster.
* The calculation happens at the client level, it retrieves the cluster state and crush rules from the monitor.

### CRUSH

* Pseudo-random placement algorithm (Fast calculation, no lookup, Repeatable, deterministic).
* Statisically distributed distribution.
* Stable mapping (Limited data migration on change).
* Rule Based configuration (Infra topology aware, adjustable replication, weghting).

