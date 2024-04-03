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
Lets you define  how you want to route your traffic using subsets and specifying appropriate labels and the service 

![alt text](<png/You, 2 days ago  1 author (You).png>)

# Virtualservice:
Lets you define how you want to route traffic to different versions using http or https

![alt text](<png/kind VirtualService.png>)

The screenshot above just show all traffic will go the v1 version. Lets run the app-01 folder to deploy the application and explore how the traffic is covered. To test, we  used a client inside kubernetes which also has a sidecar 

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

Now let's deploy another application to the production namespace making use of DNS name to access it. We will have to work on virtualservice, gateway respectively and lastly, cert-manager to secure application with TLS certificate

![Pasted Graphic 1](https://github.com/Taiwolawal/istio-project/assets/50557587/a0bc4865-fb58-48d7-b3b0-be08a91e95ac)


<img width="638" alt="created" src="https://github.com/Taiwolawal/istio-project/assets/50557587/69a2f6a4-a44b-41e6-bff3-3eedb447b0f2">

In case you do not have a domain to test with, you can use the host header format in the screenshot below

<img width="1088" alt="Pasted Graphic 51" src="https://github.com/Taiwolawal/istio-project/assets/50557587/05d6cfcf-794c-4448-a351-1ade1f09d2c1">

We can update our domain name (if you own one) with the load balancer in Route53 records

![Host name](https://github.com/Taiwolawal/istio-project/assets/50557587/9d3b1fe9-98ed-4de7-b668-126ae050ad97)

<img width="1217" alt="Pasted Graphic 52" src="https://github.com/Taiwolawal/istio-project/assets/50557587/e3dfe01d-76e0-461d-a333-2e792a6d8a32">

# Cert-Manager
We will install cert-manager and use letsencrypt to automatically obtain TLS certificates and secure our API

<img width="1246" alt="Pasted Graphic 36" src="https://github.com/Taiwolawal/istio-project/assets/50557587/a122b8ba-f624-447e-b6c6-91ecea3353f4">

To automatically obtain TLS certificate from Letsencrypt we need to creat a cluster issuer. Ensure you specify the ingress class to use solve http01 challenge

When you create these certificate, the cert-manager will obtain a certificate from letsencrypt and store it in kubernetes secret. The certificate is valid for 90 days and the cert-manager will automatically renew and update the secret

![image](https://github.com/Taiwolawal/istio-project/assets/50557587/0d06ce3d-2807-409b-b1b1-b634f3661b72)

Ensure the certificate is deployed in istio-ingress namespace where you have the gateway pod

![apiVersion cert-manager iov1](https://github.com/Taiwolawal/istio-project/assets/50557587/f7d29bb4-5d6f-4be7-b631-f66a8a028811)

<img width="736" alt="Pasted Graphic 53" src="https://github.com/Taiwolawal/istio-project/assets/50557587/66b26201-d44b-4615-ac2f-9c3cddaefe51">

Now to secure the API, we need to update the gateway file with port 443 and use https protocol and also specify the secret name that was created by the certificate resource

![- port](https://github.com/Taiwolawal/istio-project/assets/50557587/4963cabb-7a31-4879-a94e-e2633d8ba7fd)

Now let's apply the gateway file and confirm if the certificate has been issued

<img width="684" alt="Pasted Graphic 55" src="https://github.com/Taiwolawal/istio-project/assets/50557587/83f422da-993f-4354-b2cf-b9f16b7491e2">

<img width="684" alt="Pasted Graphic 55" src="https://github.com/Taiwolawal/istio-project/assets/50557587/dd63bd47-d3a3-4251-9b60-e31f098db859">

<img width="1006" alt="Pasted Graphic 58" src="https://github.com/Taiwolawal/istio-project/assets/50557587/711799c5-1bcc-44e3-9ccf-bb7b86cb7ef3">

Now lets deploy prometheus and grafana for monitoring and visualization.

<img width="844" alt="Pasted Graphic 30" src="https://github.com/Taiwolawal/istio-project/assets/50557587/48eb9f64-4dd6-4f10-ad26-9313ccb7d415">

<img width="1341" alt="Pasted Graphic 31" src="https://github.com/Taiwolawal/istio-project/assets/50557587/bfef5d12-66ec-498e-a691-23d4efab41aa">

<img width="1139" alt="Pasted Graphic 34" src="https://github.com/Taiwolawal/istio-project/assets/50557587/cbd2159a-3094-4207-b5f2-6dd546a62d03">

To monitor istio, let's create a pod monitor and use istio sidecar labels. To create a podmonitor prometheus object we need a named pod ``http-envoy-prom`` and also select the pods based on the label such as ``ìstio:monitor``. Based on this two we can start monitoring istio service mesh

<img width="698" alt="readinessProbe" src="https://github.com/Taiwolawal/istio-project/assets/50557587/d5e064b6-cb4e-42dd-92cf-4bc14656d749">

<img width="997" alt="Pasted Graphic 60" src="https://github.com/Taiwolawal/istio-project/assets/50557587/7dfee701-aa75-4c46-998c-e6021f930a48">

![kind PodMonitor](https://github.com/Taiwolawal/istio-project/assets/50557587/04665409-9097-4374-bc1d-fc23ebe08f04)

<img width="838" alt="Pasted Graphic 66" src="https://github.com/Taiwolawal/istio-project/assets/50557587/677e4a24-b62d-465d-b68a-b1b9bae7798c">

<img width="1365" alt="Pasted Graphic 67" src="https://github.com/Taiwolawal/istio-project/assets/50557587/2ea17104-b1bc-46f3-8c57-6f50dd476917">

<img width="1053" alt="Pasted Graphic 68" src="https://github.com/Taiwolawal/istio-project/assets/50557587/e6c8df19-b047-4775-97d9-bbfa3693530a">

Now lets  monitor ingress gateway also

<img width="893" alt="Pasted Graphic 69" src="https://github.com/Taiwolawal/istio-project/assets/50557587/8980ad93-bfa0-4398-9eb4-484236244725">

<img width="685" alt="Pasted Graphic 70" src="https://github.com/Taiwolawal/istio-project/assets/50557587/337ad850-18c0-41a8-95ba-5c50afe39a94">

In this case, we have a port but the name is missing so we cannot use podmonitor since we need a named port, instead we can create a service and a service monitor to target this port.

Lets define a kubernetes service that only uses prometheus port ``15090`` and give it a name ``metrics``. Now lets create a servicemonitor and use the endpoint and the metrics port name

![Pasted Graphic 3](https://github.com/Taiwolawal/istio-project/assets/50557587/4873a26b-dc9d-40ce-9af4-33526f65b906)

Now we can create a servicemonitor and use the endpoint and metrics port name, this is a useful workaround when we dont have a port name and not able to add to it but still want to monitor the application with prometheus.

![image](https://github.com/Taiwolawal/istio-project/assets/50557587/61de3913-cc3a-4dd5-b162-069432ad9131)

<img width="770" alt="192 168 16 45" src="https://github.com/Taiwolawal/istio-project/assets/50557587/18566c3f-7c65-48e6-9a0e-0b714084bb36">

<img width="439" alt="k apply -f gateway-service-monitor yaml" src="https://github.com/Taiwolawal/istio-project/assets/50557587/91b391ca-7100-4b08-80c7-2749805ce932">

<img width="1093" alt="Targets" src="https://github.com/Taiwolawal/istio-project/assets/50557587/80ad60b4-43a7-4dd0-85b4-14be564743e5">

<img width="1026" alt="Pasted Graphic 81" src="https://github.com/Taiwolawal/istio-project/assets/50557587/64ecb76b-1705-49a2-8fbc-ac6085c1a0ba">


Now lets connect to grafana

<img width="854" alt="Pasted Graphic 83" src="https://github.com/Taiwolawal/istio-project/assets/50557587/0034ad1e-664b-4254-b425-4ddf48e8ef6f">

<img width="1116" alt="Welcome to Grafana" src="https://github.com/Taiwolawal/istio-project/assets/50557587/1a219ce4-eecf-4ad5-b5b1-191e5671f34e">

Use Istio workload dashboard

<img width="1094" alt="Import dashboard" src="https://github.com/Taiwolawal/istio-project/assets/50557587/02419c91-dac4-4cf9-8f66-84f5e1e3e410">

<img width="1425" alt="Pasted Graphic 86" src="https://github.com/Taiwolawal/istio-project/assets/50557587/d2112c53-06f6-4548-b381-42036d8f879d">

<img width="1151" alt="Pasted Graphic 93" src="https://github.com/Taiwolawal/istio-project/assets/50557587/46de45fb-fd1c-4c34-81bc-8ffa535680cf">

<img width="1100" alt="Pasted Graphic 35" src="https://github.com/Taiwolawal/istio-project/assets/50557587/dbd84652-e897-48c1-80b0-d74c62226b4a">

Lets Deploy Kiali to visualize the service topology inside Kubernetes

<img width="808" alt="Pasted Graphic 87" src="https://github.com/Taiwolawal/istio-project/assets/50557587/1ef6e9d5-5795-471d-a60e-a7e22e55171b">

<img width="914" alt="Pasted Graphic 88" src="https://github.com/Taiwolawal/istio-project/assets/50557587/a928fd60-229b-4fbe-a46f-b76c37ba869c">

<img width="1424" alt="Pasted Graphic 89" src="https://github.com/Taiwolawal/istio-project/assets/50557587/6c98d3c5-c4e9-4f90-8f57-bd6cf1601f02">

<img width="1069" alt="Log in Kiali" src="https://github.com/Taiwolawal/istio-project/assets/50557587/298212ab-626b-4af6-8f3d-6869e4ef1dd9">

<img width="723" alt="Pasted Graphic 91" src="https://github.com/Taiwolawal/istio-project/assets/50557587/ba0d4a77-92f2-48ad-a4b8-8c4b7cfc7d66">

<img width="720" alt="Pasted Graphic 92" src="https://github.com/Taiwolawal/istio-project/assets/50557587/802d4042-f4cc-49ee-9cc4-36ee89678d02">


