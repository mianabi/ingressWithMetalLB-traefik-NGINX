## Deploy Ingress With MetalLB and Traefik and Nginx
## Introduction

In MetalLB, creating a Load Balancer Services Pool of IP Addresses means defining a range of IP addresses that MetalLB can assign to LoadBalancer services. This feature allows you to implement LoadBalancer services in non-cloud environments (on-premises) using fixed IP addresses or specified IP pool.
### Why do we need Pool of IP Addresses?
In cloud environments, LoadBalancer services automatically obtain external IP addresses from the cloud service provider. But in non-cloud environments, such as a local data center, these IP addresses must be managed manually. MetalLB simplifies this process and allows automatically assigning defined IPs to LoadBalancer services.
### How to define Pool of IP Addresses in MetalLB
To define an IP pool, you must create a ConfigMap for MetalLB that contains the settings for the IP pool. This ConfigMap usually contains information such as the range of IP addresses, the protocol used (such as Layer 2 or BGP), and other settings.
### Sample ConfigMap file for MetalLB
Here is an example of a ConfigMap that defines an IP pool for MetalLB:

![Screenshot from 2024-08-10 19-32-13](https://github.com/user-attachments/assets/1881c1f7-1721-49d5-b32f-9c8d75c63fc7)

In this example:

address-pools: A list of IP pools that MetalLB can use.
name: IP pool name (here "default").
protocol: the protocol used for IP assignment (here "layer2").
addresses: The range of IP addresses that will be used for LoadBalancer services.

By installing and configuring MetalLB and defining an IP pool, you can easily create LoadBalancer-type services in non-cloud environments and assign them static IP addresses. This allows you to manage your services more effectively with access to static IPs.

### Therefore, in this project, we first install and configure MetaLB to give ip to the loadBalancer service related to Traefik and Nginx Ingress Controller.

## Installation of MetalLB Load Balancer on Kubernetes Cluster

### Confirm if your Kubernetes Cluster API is responsive and you’re able to use kubectl cluster administration command-line tool:
$ kubectl cluster-info
Kubernetes control plane is running at https://k8sapi.example.com:6443
CoreDNS is running at https://k8sapi.example.com:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

### install curl and wget utilities if not available already in your workstation:
### Debian / Ubuntu ###
$ sudo apt update && sudo apt install wget curl -y

### 1. Download MetalLB installation manifest
1.1 Get the latest MetalLB release tag:

$ MetalLB_RTAG=$(curl -s https://api.github.com/repos/metallb/metallb/releases/latest|grep tag_name|cut -d '"' -f 4|sed 's/v//')

1.2 To check release tag use echo command:

$ echo $MetalLB_RTAG

1.3 Create directory where manifests will be downloaded to:

$ mkdir ~/metallb
$ cd ~/metallb

1.4 Download MetalLB installation manifest:

$ wget https://raw.githubusercontent.com/metallb/metallb/v$MetalLB_RTAG/config/manifests/metallb-native.yaml

Below are the components in the manifest file:
        ◦ The metallb-system/controller deployment – Cluster-wide controller that handles IP address assignments.
        ◦ The metallb-system/speaker daemonset – Component that speaks the protocol(s) of your choice to make the services reachable.
        ◦ Service accounts for both controller and speaker, along with RBAC permissions that the needed by the components to function.

### 2. Install MetalLB Load Balancer on Kubernetes cluster
2.1 Install MetalLB in your Kubernetes cluster by apply the manifest:

$ kubectl apply -f metallb-native.yaml

2.2 The command we executed deploys MetalLB to your Kubernetes cluster, under the metallb-system namespace.

$ watch kubectl get all -n metallb-system
$ kubectl get pods -n metallb-system --watch

2.3 Waif for everything to be in running state, then you can list running Pods:

$ kubectl get pods  -n metallb-system

2.4 To list all services instead, use the commands:

$ kubectl get all  -n metallb-system

### 3. Create Load Balancer services Pool of IP Addresses
MetalLB needs a pool of IP addresses to assign to the services when it gets such request. We have to instruct MetalLB to do so via the IPAddressPool CR.
Let’s create a file with configurations for the IPs that MetalLB uses to assign IPs to services. In the configuration the pool has IPs range 192.168.1.30-192.168.1.50.

![Screenshot from 2024-08-10 21-27-40](https://github.com/user-attachments/assets/082e131b-79f3-4f53-a332-c4c6a0d47188)

This is a sample configuration used to advertise all IP address pools created in the cluster.

![Screenshot from 2024-08-10 21-30-14](https://github.com/user-attachments/assets/f069ab3a-8948-4feb-9f08-2d951ba89e02)

A complete configuration with both IP Address Pool and L2 advertisement is shown here.

![Screenshot from 2024-08-10 21-30-47](https://github.com/user-attachments/assets/1717fb6f-6fee-4eff-b968-5f0dfe428f7b)

Apply the configuration(ipaddress_pools.yaml) using kubectl command:

$ kubectl apply -f  ~/metallb/ipaddress_pools.yaml

List created IP Address Pools and Advertisements:

![Screenshot from 2024-08-10 21-32-58](https://github.com/user-attachments/assets/ff64786a-9194-42ad-894e-6f5ea41a93d4)

Get more details using describe kubectl command option:

$ kubectl describe ipaddresspools.metallb.io production -n metallb-system

### 4. Deploying services that use MetalLB LoadBalancer()

With the MetalLB installed and configured, we can test by creating service with spec.type set to LoadBalancer, and MetalLB will do the rest. This exposes a service externally.
MetalLB attaches informational events to the services that it’s controlling. If your LoadBalancer is misbehaving, run kubectl describe service <service name> and check the event log.
We can test with this service:

$ vim web-app-demo.yaml

Paste **web-app-demo.yaml** From Files contents.

Apply configuration manifest:

$ kubectl apply -f web-app-demo.yaml
namespace/web created
deployment.apps/web-server created
service/web-server-service created

Let’s check the IP assigned by Load Balancer to the service:

![Screenshot from 2024-08-10 21-37-07](https://github.com/user-attachments/assets/83bde437-e1e6-463c-b87a-86d5ef3e3320)

Confirm the Pod is running:
$ kubectl get pod -n web

![Screenshot from 2024-08-10 21-38-40](https://github.com/user-attachments/assets/465b574b-fac5-4937-abdd-412508fce14e)

Test using telnet connectivity to the service:

$ telnet 192.168.1.30 80
Trying 192.168.1.30...
Connected to 192.168.1.30.
Escape character is '^]'.

We can also access the service from a web console via http://192.168.1.30

**Now we can raise the Traefik service on the Kubernetes cluster, and the load balancer of this service will use metalLB, its corporate IP, to provide the relevant services.**

## Install Traefik Ingress Controller

## **Step 1 – Install Traefik Ingress Controller**

### In this guide, we will install the Traefik Ingress Controller using Helm. Begin by installing Helm as below:
$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
$ chmod 700 get_helm.sh
$ sudo ./get_helm.sh

### Check your version of helm to confirm it was successfully installed:
$ helm version
version.BuildInfo{Version:"v3.10.2", GitCommit:"50f003e5ee8704ec937a756c646870227d7c8b58", GitTreeState:"clean", GoVersion:"go1.18.8"}

### Add the [Traefik Ingress helm repository](https://github.com/traefik/traefik-helm-chart) in your workstation by running the commands below:
$ helm repo add traefik https://helm.traefik.io/traefik

### Update helm charts with the commands given below:
$ helm repo update

### Create a namespace called traefik:
$ kubectl create ns traefik

### install the Traefik Ingress Controller using the helm chart on traefik namespace:
$ helm install traefik traefik/traefik --namespace traefik

### Verify the installation by checking all resources in the namespace:
$ kubectl get all -n traefik

## **Step 2 – Expose Traefik Dashboard**

### Check Traefik service to check its Load balancer IP address:
$ kubectl get svc -l app.kubernetes.io/name=traefik -n traefik

![Screenshot from 2024-08-10 19-22-35](https://github.com/user-attachments/assets/d1c17f4c-0bb8-45c5-a3c3-cfa7a5b30ce4)

### We can confirm our load balancer IP address is . You can map this in your /etc/hosts file:
$ sudo vim /etc/hosts
dev.sana.com 192.168.1.34

Now that your cluster has an IP address/domain name, you can easily access the Traefik Dashboard and web services. But currently, the service is not available since we do not have any Ingress created.
Now proceed and access the Traefik Dashboard using the http://lb_ip:9000/dashboard/

### After installing Traefik Ingress Controller on a Kubernetes cluster, the following set of objects are usually created:
**a) Namespace:**
A new namespace, usually called traefik, is created to separate resources related to Traefik Ingress Controller.

**b) Service Account:**
A service account is created for Traefik.

**c) Role and RoleBinding:** 
A Role is created to determine the necessary access for Traefik Ingress Controller. A RoleBinding is created to bind the Role to the Service Account.

**d) ClusterRole and ClusterRoleBinding:**
A ClusterRole is created to define cluster-level permissions for Traefik. A ClusterRoleBinding is created to bind the ClusterRole to the Service Account.

**e) ConfigMap:**
A ConfigMap is created to configure Traefik.

**f) Deployment:**
A Deployment is created to manage Traefik Ingress Controller pods.

**g) Service:**
A Service is created to access the Traefik Ingress Controller. This service is usually configured as LoadBalancer or NodePort.

**h) Pod:**
Deployment-related pods are created that contain Traefik Ingress Controller containers.

**i) Custom Resource Definitions (CRDs):**
A number of CRDs may be installed for more advanced features such as **IngressRoute**.
