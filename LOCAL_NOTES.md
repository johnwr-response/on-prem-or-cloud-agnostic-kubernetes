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

### Ceph with Rook
  - Rook supports all 3 types of storage: **block**, **file** and **object** storage
  - This demo will use **Ceph** for all these types, but **other backends** are also possible 
    - Rook will do a good job to **abstract this away for you** so most of the configuration is nicely hidden
    - You will be able to use the **Kubernetes yaml files** to set configuration options 
  - Deployment steps
    - First you need to deploy the **rook operator**
      - Using the provided yaml files
      - Using the Helm chart
    - Then you can **create the rook cluster**
      - Also using yaml definitions
      - Use the rook operator rather than the Kubernetes API (apiVersion: `rook.io/v1`)
    - After that, **block**/**file**/**object** storage can be configured
      - Using the rook API and Kubernetes storage API, which means that using rook storage will become as easy as using AWS EBS or NFS

### Demo: Ceph with Rook (Block storage)
- Will have four nodes
    - kubernetes-master-01
    - kubernetes-node-01
    - kubernetes-node-02
    - kubernetes-node-03
- On master:
  ````
  sudo kubectl token create --print-join-command
  kubectl get nodes
  kubectl create -f rook/rook-operator.yaml
  kubectl get pods -n rook-system
  nano rook/rook-cluster.yaml
    # change dataDirHostPath to your storage target
  kubectl create -f rook/rook-cluster.yaml
  kubectl get pods -n rook
  kubectl create -f rook/rook-storageclass.yaml
  kubectl get pods -n rook
  kubectl create -f rook/rook-tools.yaml
  kubectl get pods -n rook
  kubectl exec -it rook-tools -n rook -- bash
  rookctl status
  exit
  kubectl create -f rook/mysql-demo.yaml
  kubectl get pods
  kubectl get pv
  kubectl exec -it <mysql-pod> -- bash
  ll /var/lib/mysql
  mysql -u root -pchangeme
  create database demo; use demo; create table helloworld(id int, hello  varchar(100)); insert into helloworld values (1, 'world'); select * from helloworld;
  \q
  exit
  kubectl get pods
  kubectl describe pod/<demo-mysql-pod>
  kubectl cordon kubernetes-node-03
  kubectl get nodes
  kubectl delete pod <demo-mysql-pod>
  kubectl get pods
  kubectl describe pod/<demo-mysql-pod>
  kubectl exec -it <mysql-pod> -- bash
  mysql -u root -pchangeme
  use demo; select * from helloworld;
  \q
  exit
  kubectl get nodes
  # Shutdown node 3
  kubectl get pods -n rook
  kubectl exec -it rook-tools -n rook -- bash
  rookctl status
  exit
  ````

### Demo: Ceph with Rook (Object storage)
- Will have four nodes
    - kubernetes-master-01
    - kubernetes-node-01
    - kubernetes-node-02
    - kubernetes-node-03
- On master:
  ````
  kubectl get nodes
  kubectl uncordon kubernetes-node-03
  kubectl get nodes
  kubectl create -f rook/rook-storageclass-objectstore.yaml
  kubectl get pods -n rook
  kubectl get svc -n rook
  kubectl exec -it rook-tools -n rook -- bash
  radosgw-admin user create --uid rook-user --display-name "A rook rgw user" --rgw-realm=my-store --rgw-zonegroup=my-store
  export AWS_HOST=rook-ceph-rgw-my-store.rook
  export AWS_ENDPOINT=10.109.209.18 # (The ClusterIP of the service
  export AWS_ACCESS_KEY_ID=<ACCESS_KEY_FROM_USER_CREATE_COMMAND_ABOVE>
  export AWS_SECRET_ACCESS_KEY==<SECRET_KEY_FROM_USER_CREATE_COMMAND_ABOVE>
  s3cmd mb --no-ssl --host=${AWS_HOST} --host-bucket= s3://demobucket
  echo 'hello world' > test
  s3cmd put test --no-ssl --host=${AWS_HOST} --host-bucket= s3://demobucket
  s3cmd ls --no-ssl --host=${AWS_HOST} --host-bucket= s3://demobucket
  exit
  ````

### Demo: Ceph with Rook (File storage)
- Will have four nodes
    - kubernetes-master-01
    - kubernetes-node-01
    - kubernetes-node-02
    - kubernetes-node-03
- On master:
  ````
  kubectl create -f rook/rook-storageclass-fs.yaml
  kubectl create -f rook/fs-demo.yaml
  kubectl get pods -n rook
  kubectl get pods
  kubectl exec -it ubuntu -- bash
  mount | grep data
  ls -ahl /data
  echo 'test' > /data/helloworld 
  ls -ahl /data
  exit
  ````

# Manage TLS Certificates
## cert-manager
### Introduction
- To use a **secure http connection**, you need a **certificate**
- Certificates can be **bought** or **issued** by some **public cloud provider** like AWS certificate manager
- Managing SSL/TLS certificates yourself  often **takes a lot of time** and arre **time consuming** to **install and extend**
- You **cannot issue your own certificates** for production websites as they are not trusted by the common internet browsers
- The **cert-manager** can **ease** the **issuing** of certificates and the **management** of them
  - Can use **Let's encrypt**. A **free**, **automated** and open Certificate Authority 
    - Can issue certificates for free for your app or website
    - You will need to prove that you are the owner of a domain
    - After that they will issue a certificate for you
    - Then the certificate is **recognized by all** major software vendors and browsers
  - Cert-manager can **automate the verification process** for *let's encrypt*
  - With *Let's encrypt* you also have to **renew certificates** every **couple of months**
  - Cert-manager will **periodically check the validity** of the certificates and automatically run the **renewal process** when necessary
  - *Let's encrypt* in combination with cert-manager **takes away a lot of hassle** dealing with certificates, allowing you to **secure your endpoints** in an easy and affordable way
  - You can only issue certificates for domain names that you own (or controls)
