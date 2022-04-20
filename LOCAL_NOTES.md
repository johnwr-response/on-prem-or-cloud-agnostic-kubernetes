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

# Demo
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
 
# Operators
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
