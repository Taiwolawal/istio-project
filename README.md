# istio-project
The Project is focus on implementing different use-case for istio

For cert-maneger to work correctly `--set meshConfig.ingressSelector=gateway --set meshConfig.ingressService=istio-gateway` to be able to resolve http01 challenge from lets-encrypt


# Setup EKS Cluster
![alt text](<png/Pasted Graphic 20.png>)

![alt text](<png/Pasted Graphic 21.png>)

# Install Istio

![alt text](<png/Pasted Graphic 22.png>)

![alt text](<png/Pasted Graphic 23.png>)

# Install istiod

Ensure to update meshconfig in the istiod.yaml to allow easy installation of cert-manager so as to be able to resolve http01 challenge from lets encrypt
![alt text](global.png)

![alt text](<png/Pasted Graphic 24.png>)



![alt text](<png/Pasted Graphic 40.png>)