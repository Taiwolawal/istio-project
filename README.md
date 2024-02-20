# istio-project

The project centers around the implementation of Istio to enhance the advantages of a microservices architecture.

For cert-maneger to work correctly `--set meshConfig.ingressSelector=gateway --set meshConfig.ingressService=istio-gateway` to be able to resolve http01 challenge from lets-encrypt


# Setup EKS Cluster
![alt text](<png/Pasted Graphic 20.png>)

![alt text](<png/Pasted Graphic 21.png>)

# Install Istio

![alt text](<png/Pasted Graphic 22.png>)

![alt text](<png/Pasted Graphic 23.png>)

# Install istiod
Installation of Istiod consist of combination of pilot, citadel and gallery.
Ensure to update meshconfig in the istiod.yaml to allow easy installation of cert-manager so as to be able to resolve http01 challenge from lets encrypt

![alt text](global-2.png)

![alt text](<png/Pasted Graphic 24.png>)

Confirm if Istiod pod is running

![alt text](<Pasted Graphic 26.png>)

Now that Istio is deployed properly we can start managing traffic in the cluster using the istio control plane.

We need to ensure istio inject sidecars to all pods which enhance and control communication betweeen micro services. This injection allows Istio to provide features such as traffic management, security, observability, and policy enforcement consistently across all microservices in the cluster

Ensuring istio is deployed in all pod in a namespace, in order for istio to manage traffic, each service must have an istio sidecar

![alt text](<Pasted Graphic 2.png>)


We have two deployments to simulate canary with each one having label version "v1" and "v2"  respectively and label "app:first-app", which is used by service as the label selector and this will randomly route traffic between v1 and v2

![alt text](<Pasted Graphic 3.png>)

![alt text](<io.k8s.api.core.v1.Service (v1@service.json)  You, 2 days ago l.png>)

To handle managing and shifting traffic, we make use of some istio custom resources

# DestinationRule: 
Let you define the how you want to route your traffic using subsets and specifying appropriate labels and the service 

![alt text](<You, 2 days ago  1 author (You).png>)

# Virtualservice:
Lets you define how you want to route traffic to different versions using http or https

![alt text](<kind VirtualService.png>)

The screenshot above just show all traffic will go the v1 version. Lets run the app-01 folder to deploy the application and explore how the traffic is covered. To test, we we used a client inside kubernetes which also has a sidecar 

![alt text](<Pasted Graphic 7.png>)

![alt text](<deployment.appsfirst-app-v2 unchanged.png>)

Lets exec into the client and use curl to hit our first app service in the staging namespace. From the screenshot, we can see that we are only connecting the v1 application.

![alt text](image.png)

