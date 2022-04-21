# Installing Kubernetes 
## Kubeadm
- **Kubeadm** is a **toolkit** by Kubernetes to create a cluster
- Works on **any deb/rpm compatible OS** (Ubuntu, Debian, Redhat, CentOS, ...)
    - This is the main **advantage** of Kubeadm as **a lot of other tools are OS/Cloud specific**
- Very **easy to use** and lets you spin up your Kubernetes cluster in just a **couple of minutes** 
- Supports bootstrap tokens
  - **Simple tokens** that can be used to **create a cluster** or to **join nodes** later
  - The **tokens** are of the **format** *abcdef.0123456789abcdef*
  - Supports upgrading/downgrading clusters
  - **Does NOT install a networking solution**
    - You have to install a **C**ontainer **N**etwork **I**nterface yourself using kubectl apply
  - Prerequisites
    - dep / rpm compatible system (or CoreOS' Container Linux)
    - **2 GB** of memory
    - **2 CPUs** for the master node
    - Network connectivity between the nodes
      - Can be a private network (internal IP addresses)
      - Can be a public routable internet IP addresses (Should be firewalled)
    - Typically, you need minimum 2 nodes (One master and one node to schedule pods on) 

## Demo
- Will have two nodes
  - kubernetes-master-01
  - kubernetes-node-01
  - On master:
    ````
    scripts/install-kubernetes.sh
    scripts/create-user.sh
    ````
  - On node(s):
    ````
    scripts/install-node.sh
    kubeadm join ....
    scripts/create-user.sh #only first part
    ````
  - On master:
    ````
    kubectl get nodes
    ````
 
## Operators
- An operator is a method of **packaging**, **deploying**, and **managing** a Kubernetes Application
- It puts **operational knowledge** into an application
  - brings the user **closer to the experience of managed cloud services**, rather than having to know all the specifics of an application deployed to Kubernetes
  - Once an Operator is deployed, it can be **managed using Custom Resource Definitions** (arbitrary types that extend the Kubernetes API)
- Also provides a great way to deploy Stateful services on Kubernetes (can hide a lot of complexities from the end-user)
- Custom Resource Definitions
  - CRDs are **extensions of the Kubernetes API**
  - Allows the Kubernetes user to use **custom objects**, and CRUD those objects on the cluster
    - For example: You could run a kubectl create on  a yaml file containing a custom database object, to spin up a database on your cluster
  - The custom objects are **not necessarily available on all clusters**
    - They can be dynamically registered/deregistered
    - ****Operators include CRDs**
      - By adding an Operator, you'll register these custom resource definitions
- Operators example
  - **etcd, Rook, Prometheus and Vault**  are examples of tech that can be deployed as an Operator
  - Let's use **etcd** as an example
    - Once the etcd Operator is deployed, a new etcd cluster can be created by using the following yaml file:
    ````
    apiVersion: "etcd.database.coreos.com/v1"
    kind: "EtcdCluster"
    metadata:
      name: "example-etcs-cluster"
    spec:
      size: 3
      version: "3.2.13"
    ````
  - Resizing the cluster is  now just a matter of changing the yaml file and the run `kubectl apply`
  - The same is true for the **version** number.    
- Operators
  - Using operators **simplifies deployment and management** a lot
  - This example uses etcd, but more software is being released using operators for Kubernetes. Like:
    - The PostgreSQL operator by Zalando
      - (https://github.com/zalando-incubator/postgres-operator)
    - The MySQL operator, providing a simple API to create a MySQL database
        - (https://github.com/oracle/mysql-operator)
    ````
    apiVersion: "mysql.oracle.com/v1"
    kind: "MySQLCluster"
    metadata:
      name: "myappdb"
    ````
  - You can also build your own Operators using the following tools:
    - (https://coreos.com/operators/)
    - **The Operator SDK** makes it easy to build an operator, rather than having to learn the Kubernetes API specifics
    - **Operator Lifecycle Manager** oversees installation, updates, and management of the lifecycle of all the operators
    - **Operator Metering** usage reporting
- Operators will be used extensively here

## Rook
### Introduction
  - **Open source orchestrator** for **distributed storage systems** running in Kubernetes (https://github.com/rook/rook/tree/master/Documentation/) 
    - Allows you to **use storage systems on Kubernetes clusters**
    - If on a public cloud, it's very easy to **attach a storage volume to a pod** to persist data
  - **Rook** wants to make it easy to use a **storage system** even on on-prem clusters
  - Rook **automates** the **configuration**, **deployment**, **maintenance** of distributed storage software
  - **No need to worry** about the **difficulty of setting up storage systems**
    - Rook will **orchestrate** all this management for you
  - Currently, Rook uses **Ceph** as underlying storage, but **Minio** and **CockroachDB** are also available. **More storage engines** will be added.

### Ceph
  - **Ceph** provides **object, file and block storage**
  - Is **open source**
  - Is **distributed without a single point of failure**
  - **Replicates** it's data to make it **fault tolerant**
  - Is **self-healing** and **self-managing**
  - Is scalable to exabyte level
  - Types of storage:
    - **File storage**: to **store files and directories**. Similar to accessing files over NFS, using a NAS or using EFS on AWS
    - **Block storage**: to store data using a file system like a hard drive. Databases will typically need block storage. Similar to using a SAN or EBS on AWS.
      - A typical use case is to store files for your OS, storage for databases, etc.
    - **Object storage**: to store any type of data as an **object** identified by a **key** with the additional possibility to adding **metadata**. This type of storage lends itself to be **distributed** and **scalable**. Similar to using AWS S3.
      - Can be used to store **unstructured data** like pictures, website assets, videos, log files, etc.
  - Ceph components:
    - **Ceph monitor** (min 3): **maintains a map of the cluster state** for the other ceph components (daemons) to communicate. Is also responsible for **authentication** between daemons and clients.
    - **Ceph Manager daemon**: responsible to **keep track of runtime metrics** and the cluster state.
    - **Ceph OSDs** (Object Storage Daemon, min 3): **stores the data**. Responsible for **replication**, **recovery**, **re-balancing** and provides information for the monitoring daemons.
    - **Ceph Metadata Service** (MDSs): stores metadata for the **Ceph Filesystem** storage type (not for block/object storage types).
  - Ceph stores data as **objects** within **logical storage pools**
  - Ceph uses the **CRUSH algorithm** (Controlled Replication under Scalable Hashing) which allows Ceph to be scalable.
  - Object storage in Ceph is provided by the **distributed object storage mechanism** within Ceph
  - Ceph software libraries (librados) provides clients with access to the "Reliable Autonomic Distributed Object Store" (RADOS)  
    - RADOS provides a **reliable, autonomous, distributed object store** comprised of **self-healing, self-managing and intelligent** storage nodes
    - There is also a RESTful interface that can provide an AWS S3 compatible interface to this object store
  - Ceph **block storage** is provided by Ceph's RADOS Block Device (RBD) 
  - RBD is built on top of the same **Ceph Object Storage**
    - Ceph stores **the block images** as **objects** in the object store
    - It's also built on librados, the software library
  - Data can come in from Ceph's **file storage**, **block storage**, or **object storage** and Ceph **will store** this data as **an object**
  - Each object is stored within the Object Storage Device (OSD)
  - The OSDs wil run on multiple nodes
  - They will handle the read/write operations to their underlying storage
