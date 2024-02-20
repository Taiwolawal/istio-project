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

![alt text](<png/Pasted Graphic 40.png>)

