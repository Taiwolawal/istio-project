# istio-project

The project centers around the implementation of Istio to enhance the advantages of a microservices architecture.

# Setup EKS Cluster
![alt text](<png/Pasted Graphic 20.png>)

![alt text](<png/Pasted Graphic 21.png>)

# Install Istio

![alt text](<png/Pasted Graphic 22.png>)

![alt text](<png/Pasted Graphic 23.png>)

# Install istiod
Installation of Istiod consist of combination of pilot, citadel and gallery.
Ensure to update meshconfig in the istiod.yaml to allow easy installation of cert-manager so as to be able to resolve http01 challenge from lets encrypt


![alt text](<png/global-2.png>)

![alt text](<png/Pasted Graphic 24.png>)

Confirm if Istiod pod is running

![alt text](<png/Pasted Graphic 26.png>)

Now that Istio is deployed properly we can start managing traffic in the cluster using the istio control plane.

We need to ensure istio inject sidecars to all pods which enhance and control communication betweeen micro services. This injection allows Istio to provide features such as traffic management, security, observability, and policy enforcement consistently across all microservices in the cluster

Ensuring istio is deployed in all pod in a namespace, in order for istio to manage traffic, each service must have an istio sidecar

![alt text](<png/Pasted Graphic 2.png>)


We have two deployments to simulate canary with each one having label version "v1" and "v2"  respectively and label "app:first-app", which is used by service as the label selector and this will randomly route traffic between v1 and v2

![alt text](<png/Pasted Graphic 3.png>)

![alt text](<png/io.k8s.api.core.v1.Service (v1@service.json)  You, 2 days ago l.png>)

To handle managing and shifting traffic, we make use of some istio custom resources

# DestinationRule: 
Let you define the how you want to route your traffic using subsets and specifying appropriate labels and the service 

![alt text](<png/You, 2 days ago  1 author (You).png>)

# Virtualservice:
Lets you define how you want to route traffic to different versions using http or https

![alt text](<png/kind VirtualService.png>)

The screenshot above just show all traffic will go the v1 version. Lets run the app-01 folder to deploy the application and explore how the traffic is covered. To test, we we used a client inside kubernetes which also has a sidecar 

![alt text](<png/Pasted Graphic 7.png>)

![alt text](<png/deployment.appsfirst-app-v2 unchanged.png>)

Lets exec into the client and use curl to hit our first app service in the staging namespace. From the screenshot, we can see that we are only connecting the v1 application.

![alt text](<png/image.png>)

Now lets edit the virtual service to ensure traffic goes to v1 and v2 equally

![alt text](<png/kind VirtualService-1.png>)

![alt text](<png/virtualservice.networking.istto.tofirst-app confiqured.png>)

![alt text](<png/Pasted Graphic 46.png>)

Same with if we want all traffic to move to v2 only 

![alt text](<png/15.png>)

![alt text](<png/Pasted Graphic 47.png>)

# Istio Ingress Gateway:
We will expose application running in Kubernetes  to the internet using istio ingress gateway. Install using helm

![alt text](<png/Pasted Graphic 27.png>)

![alt text](<png/Pasted Graphic 32.png>)

![alt text](<png/Pasted Graphic 49.png>)

![alt text](<png/gateway-8698d7ddbf-16d8r.png>)

Now let's deploy another application to the production namespace making use of hostname to access it. We will have to work on virtualservice, gateway respectively and lastly, cert-manager to secure application with TLS certificate

![alt text](<png/Pasted Graphic.png>)

<img width="638" alt="created" src="https://github.com/Taiwolawal/istio-project/assets/50557587/69a2f6a4-a44b-41e6-bff3-3eedb447b0f2">

In case you do not have a domain to test with, you can use the host header format in the screenshot below

<img width="1088" alt="Pasted Graphic 51" src="https://github.com/Taiwolawal/istio-project/assets/50557587/05d6cfcf-794c-4448-a351-1ade1f09d2c1">

We can update our domain (if you own one) with the load balancer in Route53 records

![Host name](https://github.com/Taiwolawal/istio-project/assets/50557587/9d3b1fe9-98ed-4de7-b668-126ae050ad97)

<img width="1217" alt="Pasted Graphic 52" src="https://github.com/Taiwolawal/istio-project/assets/50557587/e3dfe01d-76e0-461d-a333-2e792a6d8a32">

# Cert-Manager
We will install cert-manager and use letsencrypt to automatically obtain TLS certificates and secure our API

<img width="1246" alt="Pasted Graphic 36" src="https://github.com/Taiwolawal/istio-project/assets/50557587/a122b8ba-f624-447e-b6c6-91ecea3353f4">

Ensure its deployed in istio-ingress namespace where you have the gateway pod

![apiVersion cert-manager iov1](https://github.com/Taiwolawal/istio-project/assets/50557587/f7d29bb4-5d6f-4be7-b631-f66a8a028811)

When you create these certificate, the cert-manager will obtain a certificate from letsencrypt and store it in kubernetes secret. The certificate is valid for 90 days and the cert-manager will automatically renew and update the secret

<img width="736" alt="Pasted Graphic 53" src="https://github.com/Taiwolawal/istio-project/assets/50557587/66b26201-d44b-4615-ac2f-9c3cddaefe51">


