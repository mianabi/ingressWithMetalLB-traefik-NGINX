## Deploye and test Traefik Ingress Route
After we upload Traefik on the Kuber cluster, to test whether the traffic is actually passing through Traefik and to serve a simple web, we need to run the following files in order: 
1- run Deployement Workload for up pod and Clusterip Service by apply mysite.yaml:

$ kubectl apply -f mysite.yaml -n traefik

$ kubectl get svc -n traefik

Now, this application is still not accessible until you create an Ingress to expose the service.
Add the below lines and replace the host with the domain name added in /etc/hosts for the external IP address.

2- Apply ingress.yaml and ingressRoute.yaml:

$ kubectl apply -f ingress.yaml -n traefik

$ kubectl apply -f ingressRoute.yaml -n traefik

Now access the deployment using the URL http://domain_name
