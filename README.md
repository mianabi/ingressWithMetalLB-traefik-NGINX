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
### Namespace: 
A new namespace, usually called traefik, is created to separate resources related to Traefik Ingress Controller.
### Service Account:
A service account is created for Traefik.
### Role and RoleBinding: 
A Role is created to determine the necessary access for Traefik Ingress Controller. A RoleBinding is created to bind the Role to the Service Account.
### ClusterRole and ClusterRoleBinding: 
A ClusterRole is created to define cluster-level permissions for Traefik. A ClusterRoleBinding is created to bind the ClusterRole to the Service Account.
### ConfigMap: 
A ConfigMap is created to configure Traefik.
### Deployment: 
A Deployment is created to manage Traefik Ingress Controller pods.
### Service: 
A Service is created to access the Traefik Ingress Controller. This service is usually configured as LoadBalancer or NodePort.
### Pod: 
Deployment-related pods are created that contain Traefik Ingress Controller containers.
### Custom Resource Definitions (CRDs): 
A number of CRDs may be installed for more advanced features such as **IngressRoute**.
