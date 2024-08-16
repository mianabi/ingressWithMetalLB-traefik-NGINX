## Deploy and test NGINX Ingress Route
this Files deploye 3 pod for Serve Wen Site on Nginx and route Traefik by Nginx Ingress route: 

1- Create namespace "dev" and run Deployement Workload for up pod by apply deployement.yaml on it:

$ kubectl create ns dev
$ kubectl apply -f mysite.yaml -n dev

$ kubectl get pod -n dev

Now, this application is still not accessible until you create an Ingress to expose the service.
Add the below lines and replace the host with the domain name added in /etc/hosts for the external IP address.
2- for view pods from out of Cluster kuber apply clusterip.yaml to create clusterip Service for Pods:

$ kubectl apply -f clusterip.yaml -n dev
$ kubectl get svc -n dev

3- Apply ingress.yaml manifest for route trafik to Service and Pods by Namespace,Port and Label:
$ kubectl apply -f ingress.yaml -n dev

Now access the deployment using the URL http://domain_name
