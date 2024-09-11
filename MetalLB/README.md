## Installation of MetalLB Load Balancer on Kubernetes Cluster

### Confirm if your Kubernetes Cluster API is responsive and you’re able to use kubectl cluster administration command-line tool:
```yml
$ kubectl cluster-info

Kubernetes control plane is running at https://k8sapi.example.com:6443
CoreDNS is running at https://k8sapi.example.com:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```


### install curl and wget utilities if not available already in your workstation:
### Debian / Ubuntu ###
```yml
$ sudo apt update && sudo apt install wget curl -y
```

### 1. Download MetalLB installation manifest
1.1 Get the latest MetalLB release tag:
```yml
$ MetalLB_RTAG=$(curl -s https://api.github.com/repos/metallb/metallb/releases/latest|grep tag_name|cut -d '"' -f 4|sed 's/v//')
```

1.2 To check release tag use echo command:
```yml
$ echo $MetalLB_RTAG
```

1.3 Create directory where manifests will be downloaded to:
```yml
$ mkdir ~/metallb
$ cd ~/metallb
```

1.4 Download MetalLB installation manifest:
```yml
$ wget https://raw.githubusercontent.com/metallb/metallb/v$MetalLB_RTAG/config/manifests/metallb-native.yaml
```

Below are the components in the manifest file:
        ◦ The metallb-system/controller deployment – Cluster-wide controller that handles IP address assignments.
        ◦ The metallb-system/speaker daemonset – Component that speaks the protocol(s) of your choice to make the services reachable.
        ◦ Service accounts for both controller and speaker, along with RBAC permissions that the needed by the components to function.

### 2. Install MetalLB Load Balancer on Kubernetes cluster
2.1 Install MetalLB in your Kubernetes cluster by apply the manifest:
```yml
$ kubectl apply -f metallb-native.yaml
```

2.2 The command we executed deploys MetalLB to your Kubernetes cluster, under the metallb-system namespace.
```yml
$ watch kubectl get all -n metallb-system
$ kubectl get pods -n metallb-system --watch
```

2.3 Waif for everything to be in running state, then you can list running Pods:
```yml
$ kubectl get pods  -n metallb-system
```

2.4 To list all services instead, use the commands:
```yml
$ kubectl get all  -n metallb-system
```

### 3. Create Load Balancer services Pool of IP Addresses
MetalLB needs a pool of IP addresses to assign to the services when it gets such request. We have to instruct MetalLB to do so via the IPAddressPool CR.
Let’s create a file with configurations for the IPs that MetalLB uses to assign IPs to services. In the configuration the pool has IPs range 192.168.1.30-192.168.1.50.

![Screenshot from 2024-08-10 21-27-40](https://github.com/user-attachments/assets/082e131b-79f3-4f53-a332-c4c6a0d47188)

This is a sample configuration used to advertise all IP address pools created in the cluster.

![Screenshot from 2024-08-10 21-30-14](https://github.com/user-attachments/assets/f069ab3a-8948-4feb-9f08-2d951ba89e02)

A complete configuration with both IP Address Pool and L2 advertisement is shown here.

![Screenshot from 2024-08-10 21-30-47](https://github.com/user-attachments/assets/1717fb6f-6fee-4eff-b968-5f0dfe428f7b)

Apply the configuration(ipaddress_pools.yaml) using kubectl command:
```yml
$ kubectl apply -f  ~/metallb/ipaddress_pools.yaml
```

List created IP Address Pools and Advertisements:

![Screenshot from 2024-08-10 21-32-58](https://github.com/user-attachments/assets/ff64786a-9194-42ad-894e-6f5ea41a93d4)

Get more details using describe kubectl command option:
```yml
$ kubectl describe ipaddresspools.metallb.io production -n metallb-system
```

### 4. Deploying services that use MetalLB LoadBalancer()

With the MetalLB installed and configured, we can test by creating service with spec.type set to LoadBalancer, and MetalLB will do the rest. This exposes a service externally.
MetalLB attaches informational events to the services that it’s controlling. If your LoadBalancer is misbehaving, run kubectl describe service <service name> and check the event log.
We can test with this service:
```yml
$ vim web-app-demo.yaml
```

Paste **web-app-demo.yaml** From Files contents.

Apply configuration manifest:
```yml
$ kubectl apply -f web-app-demo.yaml
```
namespace/web created
deployment.apps/web-server created
service/web-server-service created

Let’s check the IP assigned by Load Balancer to the service:

![Screenshot from 2024-08-10 21-37-07](https://github.com/user-attachments/assets/83bde437-e1e6-463c-b87a-86d5ef3e3320)

Confirm the Pod is running:
```yml
$ kubectl get pod -n web
```

![Screenshot from 2024-08-10 21-38-40](https://github.com/user-attachments/assets/465b574b-fac5-4937-abdd-412508fce14e)

Test using telnet connectivity to the service:
```yml
$ telnet 192.168.1.30 80
```
Trying 192.168.1.30...
Connected to 192.168.1.30.
Escape character is '^]'.

We can also access the service from a web console via http://192.168.1.30

**Now we can raise the Traefik service on the Kubernetes cluster, and the load balancer of this service will use metalLB, its corporate IP, to provide the relevant services.**

