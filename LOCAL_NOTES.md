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
