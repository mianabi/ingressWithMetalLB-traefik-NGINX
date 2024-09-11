## Deploy Ingress With MetalLB and Traefik and Nginx
## Introduction

In MetalLB, creating a Load Balancer Services Pool of IP Addresses means defining a range of IP addresses that MetalLB can assign to LoadBalancer services. This feature allows you to implement LoadBalancer services in non-cloud environments (on-premises) using fixed IP addresses or specified IP pool.
### Why do we need Pool of IP Addresses?
In cloud environments, LoadBalancer services automatically obtain external IP addresses from the cloud service provider. But in non-cloud environments, such as a local data center, these IP addresses must be managed manually. MetalLB simplifies this process and allows automatically assigning defined IPs to LoadBalancer services.
### How to define Pool of IP Addresses in MetalLB
To define an IP pool, you must create a ConfigMap for MetalLB that contains the settings for the IP pool. This ConfigMap usually contains information such as the range of IP addresses, the protocol used (such as Layer 2 or BGP), and other settings.
### Sample ConfigMap file for MetalLB
Here is an example of a ConfigMap that defines an IP pool for MetalLB:

```yml
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 192.168.1.240-192.168.1.250
```

In this example:

address-pools: A list of IP pools that MetalLB can use.
name: IP pool name (here "default").
protocol: the protocol used for IP assignment (here "layer2").
addresses: The range of IP addresses that will be used for LoadBalancer services.

By installing and configuring MetalLB and defining an IP pool, you can easily create LoadBalancer-type services in non-cloud environments and assign them static IP addresses. This allows you to manage your services more effectively with access to static IPs.

### **Step 1 –** Therefore, in this project, we first install and configure MetaLB to give ip to the loadBalancer service related to Traefik and Nginx Ingress Controller.

### **Step 2 –** Run manifests in Nginx Folder and test Nginx Ingress Controller.

### **Step 3 –** install Traefik and Run manifests in Traefik Folder anf Test Simple nginx Web Site and it's Traefik.
