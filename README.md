# K8S-Installation-and-setup on Ubuntu
#  **<span style="color:green">Sani Solutions.</span>**
### **<span style="color:green">Contacts: +45 715 730 47</span>**
### **Email: forchu.cha@gmail.com**


## Kubernetes Setup Using Kubeadm In AWS EC2 Ubuntu Servers.
##### Prerequisite
+ AWS Acccount.
+ Create 3 - Ubuntu Servers -- 18.04.
+ 1 Master (4GB RAM , 2 Core)  t2.medium
+ 2 Workers  (1 GB, 1 Core)     t2.micro
+ Create Security Group and open required ports for kubernetes.
   + Open all port for this illustration
+ Attach Security Group to EC2 Instance/nodes.

## Assign hostname &  login as ‘root’ user because the following set of commands need to be executed with ‘sudo’ permissions.
```sh
sudo hostnamectl set-hostname master
sudo -i
```
```sh
apt-get update -y
apt-get install ca-certificates curl gnupg lsb-release -y
```
#Note: We are not installing Docker Here. Since containerd.io package is part of docker apt repositories hence we added the docker repository & its key to download and install containerd.
# Add Docker’s official GPG key:
```sh
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```
#Use follwing command to set up the repository:
```sh
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
# Install containerd

```sh
apt-get update -y
apt-get install containerd.io -y
```
# Generate default configuration file for containerd

#Note: Containerd uses a configuration file located in /etc/containerd/config.toml for specifying daemon level options.
#The default configuration can be generated via below command.
```sh
containerd config default > /etc/containerd/config.toml
```
# Run following command to update configure cgroup as systemd for contianerd.
```sh
sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
```
# Restart and enable containerd service
```sh
systemctl restart containerd
systemctl enable containerd
```
#5) Installing kubeadm, kubelet and kubectl

# Update the apt package index and install packages needed to use the Kubernetes apt repository:
```sh
apt-get update
apt-get install -y apt-transport-https ca-certificates curl
```
# Download the Google Cloud public signing key:
```sh
curl -fsSLo /etc/apt/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
```
# Add the Kubernetes apt repository:
```sh
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
# Update apt package index, install kubelet, kubeadm, and kubectl, and pin their version:
```sh
apt-get update
apt-get install -y kubelet kubeadm kubectl
```
# apt-mark hold will prevent the package from being automatically upgraded or removed.
```sh
apt-mark hold kubelet kubeadm kubectl
```
# Enable and start kubelet service
```sh
systemctl daemon-reload
systemctl start kubelet
systemctl enable kubelet.service
```

# Exit as root user & execute the below commands as normal Ubuntu user
```sh
sudo su - ubuntu
```
# Initialised the control plane.
# Initialize Kubernetes master by executing the below command.
```sh
sudo kubeadm init
```
```sh
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
To verify, if kubectl is working or not, run the following command.
kubectl get pods -A
```
# deploy the network plugin - weave network
```sh
kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
kubectl get pods -A
kubectl get node 
```
# Copy kubeadm join token from the master and execute in Worker Nodes to join to cluster
```sh
kubeadm join 172.31.10.12:6443 --token cdm6fo.dhbrxyleqe5suy6e \
        --discovery-token-ca-cert-hash sha256:1fc51686afd16c46102c018acb71ef9537c1226e331840e7d401630b96298e7d

```

kubeadm join 172.31.27.203:6443 --token b6aelh.35v91d8r8r40u1se --discovery-token-ca-cert-hash sha256:03958713c0774924c4da2294e1dc8309cf98cbfd7b528f6250b530bc6cb12145



K8s resources used to deploy application:
========================================

1. Pods or ControllerManager:
  Containers run in pods
  controller managers include: replica set/Daemonset/statefulset/Deployment/Colume/Job
2. applications in K8s can be accessed using Service Discovery
     Service types include
     ClusterIP
     NodePort
     LoadBalancer
All service types perform LB. Service does DNS resolution (call the service and it will route traffic to the pods)


# Where Deployment Takes Place in K8s:

Deployment takes place in a NameSpace. It is a virtual cluster in a Cluster. 
It is used to isolate different environments (prod, dev, test), sales/account/payroll
It is also used for permissions(dev, engineers)/ resource utilization 9Gi/ Performance (priority)

Kubernetes uses Kubectl client or UI to run a workload

kubectl get ns /  -n: to specify
NAME              STATUS   AGE
default           Active   20h   === App will be deployed in the default ns if not specified
kube-node-lease   Active   20h
kube-public       Active   20h
kube-system       Active   20h   === Contains system static pods created when nodes were created

kubectl create ns <NameSpaceName>

Declarative Approach
=====================
apiVersion: V1
Kind: Namespace
metadata:
  name: prod
```sh
kubectl API-resources | grep namespace:  to get API version
```
PODS:
=====

pod is the smallest building block in which app are deployed in k8s.
pods can be deployed in a NameSpace pod can contain 2 or more containers called multi pod Containers (logmnt or utility) or single pod container 
All containers in a pod share the same network, and storage. Pods have unique IP Addresses in a cluster 
containers in the same pod share the same network, filesystem, and storage and can communicate using 
localhost:<containerport>

How to deploy Applications in Pods:
==================================

```sh
kubectl run <podName> --image=<ImageName> --port=<containerport> -n <NameSpace>

kubectl run hello --image=mylandmarktech/hello --port=80 -n dev
```
Images are pulled from DockerHub or other registries like Nexus
For Best practice, app are deployed using manifest files.
App is deployed in an isolated envnt called NameSpace using Manifest files 


pod.yml
==========
```sh
kind: Pod 
apiVersion: v1 
metadata:
  name: myapp
  namespace: dev
  labels:
    app: myapp
    tier: fe
spec:
  containers:
    -name: webapp
    image: mylandmarktech/hello
    ports:
    -containerPort: 80
```
# Run the command to apply
```sh
kubectl apply -f pod.yml 
kubectl get pod -n dev 
```
the current context is default NameSpace
to change, run 
```sh
kubectl config set-context --current --namespace=<DesiredNameSpace>
```
Application Discovery:
=====================

Applications are discovered in pods using Service

service.yml
============
```sh
kind: Service
apiVersion: v1 
metadata:
  name: myappsvc
  namespace: dev
spec:
  type: ClusterIP
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: myapp 
```
ClusterIP service is the default service type. It is communication between app in the cluster.

name is the service name
namespace is the app/pod namespace
selector must be equal to labels(appName)
port is the service pods
target port is the container port 


kubectl get svc:
NAME       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
myappsvc   ClusterIP   10.100.232.162   <none>        80/TCP    15s
kubectl get ep:
NAME       ENDPOINTS        AGE
myappsvc   10.44.0.1:8080   61s

service is port is 80 and it is routing traffic to myapp on endpoint 8080

```sh
kubectl get all:  app pods and services created
kubectl delete pod --all
kubectl get pods
kubectl get pods -o yaml
kubectl get pods -o wide
kubectl get pods --show-labels
kubectl describe pod <podName>
kubectl describe svc 
kubectl delete svc 
```
curl pod IP:podPortnumber
       podIP    ContainerPort
curl 10.44.0.1:8080
        SVCIP      cIP    appPath  
curl -v 10.44.0.1:8080/java-web-app


service makes pods accesible and discoverable from within and outside the Cluster
service identify pods using labels and selector
when a service is created, a ClusterIP is allocated and DNS entries is created for that IP
service name does DNS resolution.

STATIC PODS IN K8S:
===================

they are pods controlled by the kubelet service
Kubernetes has self-healing capabilities
PODS in k8s have a short lifespan. if a node goes does on which a pod is running, the pod will not be recreated:
IF pods are created with controller managers, the pod lifecycle can be managed

sudo vi /etc/kubernetes/manifsest/file.yml:  create a managed pod for replication
the pod is controlled by the kubelet service
```sh
kind: Pod 
apiVersion: v1 
metadata:
  name: webapp
  namespace: dev
  labels:
    app: webapp
spec:
  containers:
  - name: app
    image: mylandmarktech/maven-web-app
    ports:
    - containerPort: 8080
```
if components of the static pods are deleted, it will be recreated by the kubelet service


For end users to access the application, NodePort Service is required.

          SVCIP          port     app
curl -v 18.118.206.156:31771/java-web-app









