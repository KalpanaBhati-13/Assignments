**1.Create a Kubernetes cluster using minikube.**

To create a Kubernetes cluster using Minikube, follow these step-by-step instructions. This guide assumes you are using Windows, macOS, or Linux with admin privileges.



Step 1: Install Prerequisites

1\. Install a Hypervisor or Container Driver

Windows/macOS: Use Docker, VirtualBox, Hyper-V, etc.



Linux: Docker or VirtualBox recommended.



Install kubectl (Kubernetes CLI)

On macOS:

brew install kubectl

On Windows (with Chocolatey):



powershell

choco install kubernetes-cli

On Linux:

sudo apt install -y kubectl

Or use the official installation docs: https://kubernetes.io/docs/tasks/tools/install-kubectl/



Step 2: Install Minikube

On macOS:

brew install minikube

On Windows (with Chocolatey):



powershell

choco install minikube

On Linux:

curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64

sudo install minikube-linux-amd64 /usr/local/bin/minikube



Step 3: Start a Minikube Cluster

You can start a basic cluster using:

minikube start

If you're using Docker as the driver:

minikube start --driver=docker

You can specify the Kubernetes version (optional):

minikube start --kubernetes-version=v1.29.2



Step 4: Verify the Cluster

Check the status:

minikube status

Check nodes:

kubectl get nodes



Step 5: Use Your Cluster

You can now deploy apps to your cluster. Example:

kubectl create deployment hello-minikube --image=kicbase/echo-server:1.0

kubectl expose deployment hello-minikube --type=NodePort --port=8080

minikube service hello-minikube

Optional: Enable Minikube Dashboard

bash

Copy

Edit

minikube dashboard

This opens a GUI in your browser.



To Stop or Delete the Cluster

Stop:

minikube stop

Delete:

minikube delete.



**2.Create a Kubernetes cluster using kubeadm.**

Creating a Kubernetes cluster using kubeadm gives you more control than Minikube, and is closer to production setups. Below is a step-by-step guide to create a basic Kubernetes cluster (1 master, 1+ worker nodes) using kubeadm.



Prerequisites

2 or more Linux machines (VMs or physical)



1 master node



1 or more worker nodes



OS: Ubuntu 20.04+ / CentOS 7+ recommended



Minimum 2 CPUs, 2GB RAM per node



Full network connectivity between nodes



Swap must be disabled on all nodes



Step 1: Setup on All Nodes

Run these on all nodes (master and workers):



Disable swap:

sudo swapoff -a

sudo sed -i '/ swap / s/^/#/' /etc/fstab

2\. Enable required kernel modules and settings:

cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf

overlay

br\_netfilter

EOF



sudo modprobe overlay

sudo modprobe br\_netfilter



cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf

net.bridge.bridge-nf-call-iptables  = 1

net.ipv4.ip\_forward                 = 1

net.bridge.bridge-nf-call-ip6tables = 1

EOF



sudo sysctl --system

Step 2: Install container runtime (Containerd)

sudo apt-get update

sudo apt-get install -y containerd



\#Create default config

sudo mkdir -p /etc/containerd

containerd config default | sudo tee /etc/containerd/config.toml



\# Restart containerd

sudo systemctl restart containerd

sudo systemctl enable containerd

Step 3: Install kubeadm, kubelet, and kubectl

sudo apt-get update

sudo apt-get install -y apt-transport-https ca-certificates curl



sudo curl -fsSLo /etc/apt/trusted.gpg.d/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg



echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | \\

&nbsp; sudo tee /etc/apt/sources.list.d/kubernetes.list



sudo apt-get update

sudo apt-get install -y kubelet kubeadm kubectl

sudo apt-mark hold kubelet kubeadm kubectl



Step 4: Initialize Kubernetes Master (only on master node)

sudo kubeadm init --pod-network-cidr=192.168.0.0/16

Note: --pod-network-cidr is needed for network plugins like Calico/Flannel.



After completion, follow the instructions shown in the terminal output — typically:

mkdir -p $HOME/.kube

sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

sudo chown $(id -u):$(id -g) $HOME/.kube/config



Step 5: Install a Pod Network (CNI Plugin)

Example using Calico:

kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/calico.yaml



Step 6: Join Worker Nodes

On each worker node, use the kubeadm join command that was shown at the end of the master init step, e.g.



sudo kubeadm join 192.168.1.100:6443 --token abcdef.0123456789abcdef \\

&nbsp;   --discovery-token-ca-cert-hash sha256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx



Step 7: Verify the Cluster

On the master node, run:

kubectl get nodes

You should see the master and all joined worker nodes.



Optional: Reset a Node

If you want to reset and start again:

sudo kubeadm reset

sudo rm -rf ~/.kube.



**3.Deploy an AKS cluster using the portal. Access the dashboard and create roles for multiple users.**

PART 1: Deploy AKS Cluster Using Azure Portal

Sign in to Azure Portal



Search for "Kubernetes services" in the top search bar, and click “+ Create”



Configure the cluster basics:



Subscription: Your Azure subscription



Resource Group: Create a new or use existing one



Cluster Name: Example myAKSCluster



Region: Choose your preferred Azure region



Kubernetes Version: Choose default/stable



Node Pool Configuration:



Node size: e.g. Standard\_DS2\_v2



Node count: e.g. 1–3 nodes



Authentication:



Choose System-assigned managed identity



Enable RBAC (Role-Based Access Control)



Networking:



Choose Kubenet or Azure CNI (default is fine)



Click "Review + create", then Create



Wait a few minutes for the deployment to complete.



PART 2: Access the Kubernetes Dashboard

The AKS dashboard is not enabled by default. You access it through kubectl proxy.



1\. Install Azure CLI and kubectl

If not already installed:

az login

az aks install-cli



2\. Get Cluster Credentials

az aks get-credentials --resource-group <your-resource-group> --name <your-cluster-name>

Example:

az aks get-credentials --resource-group myResourceGroup --name myAKSCluster



3\. Deploy Kubernetes Dashboard

kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml



4\. Start the Dashboard Proxy

kubectl proxy

Open in browser:

http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/

PART 3: Create Roles for Multiple Users

Step 1: Create a ServiceAccount

yaml

apiVersion: v1

kind: ServiceAccount

metadata:

&nbsp; name: dev-user

&nbsp; namespace: default

Apply it:

kubectl apply -f dev-user.yaml



Step 2: Create a RoleBinding (for namespace-scoped access)

yaml

apiVersion: rbac.authorization.k8s.io/v1

kind: RoleBinding

metadata:

&nbsp; name: dev-user-view

&nbsp; namespace: default

subjects:

\- kind: ServiceAccount

&nbsp; name: dev-user

&nbsp; namespace: default

roleRef:

&nbsp; kind: ClusterRole

&nbsp; name: view

&nbsp; apiGroup: rbac.authorization.k8s.io

Apply it:

kubectl apply -f dev-user-rolebinding.yaml

Replace view with edit or admin for higher privileges.



Step 3: Get Access Token for Login

Run:

kubectl -n default describe secret $(kubectl -n default get secret | grep dev-user | awk '{print $1}')

Use the token shown to log in to the Kubernetes dashboard using the "Token" option.



Optional: Add Azure AD User Access (for enterprise)

If you're using Azure AD-integrated AKS, you can assign cluster roles to AAD users like so:

kubectl create clusterrolebinding <binding-name> \\

&nbsp; --clusterrole=cluster-admin \\

&nbsp; --user=<aad-user-object-id-or-UPN>.



**4.Deploy a microservice application on AKS cluster and access it using public internet.**

Prerequisites

Ensure the following are installed:



Azure CLI (az)



kubectl



You have an AKS cluster deployed and configured:

az aks get-credentials --resource-group <your-rg> --name <your-aks-cluster>

PART 1: Sample Microservice App (Frontend + Backend)

We'll deploy:



A simple backend API (e.g., Node.js or Python)



A frontend service that calls the backend



Expose the frontend to the internet via LoadBalancer



Directory Structure (optional for local files)

/microservice-app/

├── backend-deployment.yaml

├── backend-service.yaml

├── frontend-deployment.yaml

├── frontend-service.yaml



PART 2: Deploy the Backend

backend-deployment.yaml

yaml

apiVersion: apps/v1

kind: Deployment

metadata:

&nbsp; name: backend

spec:

&nbsp; replicas: 2

&nbsp; selector:

&nbsp;   matchLabels:

&nbsp;     app: backend

&nbsp; template:

&nbsp;   metadata:

&nbsp;     labels:

&nbsp;       app: backend

&nbsp;   spec:

&nbsp;     containers:

&nbsp;     - name: backend

&nbsp;       image: hashicorp/http-echo

&nbsp;       args:

&nbsp;         - "-text=Hello from backend!"

&nbsp;       ports:

&nbsp;       - containerPort: 5678

backend-service.yaml

yaml

apiVersion: v1

kind: Service

metadata:

&nbsp; name: backend-service

spec:

&nbsp; selector:

&nbsp;   app: backend

&nbsp; ports:

&nbsp;   - protocol: TCP

&nbsp;     port: 80

&nbsp;     targetPort: 5678

&nbsp; type: ClusterIP



PART 3: Deploy the Frontend

frontend-deployment.yaml

yaml

apiVersion: apps/v1

kind: Deployment

metadata:

&nbsp; name: frontend

spec:

&nbsp; replicas: 2

&nbsp; selector:

&nbsp;   matchLabels:

&nbsp;     app: frontend

&nbsp; template:

&nbsp;   metadata:

&nbsp;     labels:

&nbsp;       app: frontend

&nbsp;   spec:

&nbsp;     containers:

&nbsp;     - name: frontend

&nbsp;       image: nginx

&nbsp;       ports:

&nbsp;       - containerPort: 80

(In a real-world app, you'd use a frontend image that makes API calls to backend-service.)



frontend-service.yaml

yaml

apiVersion: v1

kind: Service

metadata:

&nbsp; name: frontend-service

spec:

&nbsp; selector:

&nbsp;   app: frontend

&nbsp; ports:

&nbsp;   - protocol: TCP

&nbsp;     port: 80

&nbsp; type: LoadBalancer



PART 4: Apply the YAML Files

Apply them using kubectl:

kubectl apply -f backend-deployment.yaml

kubectl apply -f backend-service.yaml

kubectl apply -f frontend-deployment.yaml

kubectl apply -f frontend-service.yaml



PART 5: Access App via Public IP

Run:

kubectl get service frontend-service

You’ll see output like:



pgsql

NAME              TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)        AGE

frontend-service  LoadBalancer   10.0.180.12    52.183.19.234     80:31145/TCP   1m

Open the EXTERNAL-IP in your browser:



cpp

http://52.183.19.234

frontend app (like NGINX default page).



Optional: Add Ingress Controller \& TLS

For production:



Use NGINX Ingress Controller



Setup TLS/SSL with cert-manager



Use DNS instead of IP.



**5.Expose services in the cluster with node port, cluster IP, load balancer.**

In Kubernetes, services expose applications running in Pods in different ways depending on the use case. Let's walk through how to expose services using:



NodePort



ClusterIP



LoadBalancer



1\. ClusterIP (Default, Internal)

Use Case: Internal communication between services (e.g., frontend → backend).



Access: Only inside the cluster.



Example:

yaml

apiVersion: v1

kind: Service

metadata:

&nbsp; name: my-backend

spec:

&nbsp; selector:

&nbsp;   app: backend

&nbsp; ports:

&nbsp;   - protocol: TCP

&nbsp;     port: 80        # exposed internally

&nbsp;     targetPort: 8080  # pod's port

&nbsp; type: ClusterIP

Pods can access it with:

http://my-backend:80 from the same namespace.



2\. NodePort (Public on Node IP + Port)

Use Case: Expose service outside the cluster using <NodeIP>:<NodePort>.



Access: External (public IP of any worker node + port).



Example:

yaml

apiVersion: v1

kind: Service

metadata:

&nbsp; name: my-nodeport-service

spec:

&nbsp; selector:

&nbsp;   app: myapp

&nbsp; ports:

&nbsp;   - protocol: TCP

&nbsp;     port: 80

&nbsp;     targetPort: 8080

&nbsp;     nodePort: 30036  # optional, Kubernetes assigns if not set

&nbsp; type: NodePort

Access it from a browser or curl:



http://<any-node-ip>:30036

Port range: 30000–32767



3\. LoadBalancer (Cloud Provider Only – AKS, EKS, GKE)

Use Case: Public internet access via a cloud load balancer.



Access: External IP or DNS provided by cloud provider.



Example:

yaml

apiVersion: v1

kind: Service

metadata:

&nbsp; name: my-loadbalancer-service

spec:

&nbsp; selector:

&nbsp;   app: myapp

&nbsp; ports:

&nbsp;   - protocol: TCP

&nbsp;     port: 80

&nbsp;     targetPort: 8080

&nbsp; type: LoadBalancer

In AKS, Azure will automatically provision a public IP.

Check it with:

kubectl get service my-loadbalancer-service

Then access via:

http://<external-ip>:80

Summary Table

Type			Use Case

ClusterIP		Internal service-to-service

NodePort	 (node IP + port)	Quick external access for dev

LoadBalancer	 (via cloud)	Public-facing services











