## Install Traefik Ingress Controller

## **Step 1 – Install Traefik Ingress Controller**

### In this guide, we will install the Traefik Ingress Controller using Helm. Begin by installing Helm as below:
```yml
$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
$ chmod 700 get_helm.sh
$ sudo ./get_helm.sh
```

### Check your version of helm to confirm it was successfully installed:
```yml
$ helm version
```
version.BuildInfo{Version:"v3.10.2", GitCommit:"50f003e5ee8704ec937a756c646870227d7c8b58", GitTreeState:"clean", GoVersion:"go1.18.8"}

### Add the [Traefik Ingress helm repository](https://github.com/traefik/traefik-helm-chart) in your workstation by running the commands below:
```yml
$ helm repo add traefik https://helm.traefik.io/traefik
```

### Update helm charts with the commands given below:
```yml
$ helm repo update
```

### Create a namespace called traefik:
```yml
$ kubectl create ns traefik
```

### install the Traefik Ingress Controller using the helm chart on traefik namespace:
```yml
$ helm install traefik traefik/traefik --namespace traefik
```

### Verify the installation by checking all resources in the namespace:
```yml
$ kubectl get all -n traefik
```

## **Step 2 – Expose Traefik Dashboard**

### Check Traefik service to check its Load balancer IP address:
```yml
$ kubectl get svc -l app.kubernetes.io/name=traefik -n traefik
```

![Screenshot from 2024-08-10 19-22-35](https://github.com/user-attachments/assets/d1c17f4c-0bb8-45c5-a3c3-cfa7a5b30ce4)

### We can confirm our load balancer IP address is . You can map this in your /etc/hosts file:
```yml
$ sudo vim /etc/hosts
dev.sana.com 192.168.1.34
```

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

## **Step 3 –** Deploy and test Traefik Ingress Route
After we upload Traefik on the Kuber cluster, to test whether the traffic is actually passing through Traefik and to serve a simple web, we need to run the following files in order: 

1- run Deployement Workload for up pod and Clusterip Service by apply mysite.yaml:
```yml
$ kubectl apply -f mysite.yaml -n traefik
$ kubectl get svc -n traefik
```

Now, this application is still not accessible until you create an Ingress to expose the service.
Add the below lines and replace the host with the domain name added in /etc/hosts for the external IP address.

2- Apply ingress.yaml and ingressRoute.yaml:
```yml
$ kubectl apply -f ingress.yaml -n traefik
$ kubectl apply -f ingressRoute.yaml -n traefik
```

Now access the deployment using the URL http://domain_name
