**1.Deploy Replica Set and Replication Controller, and deployment. Also learn the advantages and disadvantages of each.**

1\. ReplicaSet

ReplicaSet ensures a specified number of pod replicas are running at all times.



YAML for ReplicaSet:

yaml

apiVersion: apps/v1

kind: ReplicaSet

metadata:

&nbsp; name: my-replicaset

spec:

&nbsp; replicas: 3

&nbsp; selector:

&nbsp;   matchLabels:

&nbsp;     app: myapp

&nbsp; template:

&nbsp;   metadata:

&nbsp;     labels:

&nbsp;       app: myapp

&nbsp;   spec:

&nbsp;     containers:

&nbsp;     - name: mycontainer

&nbsp;       image: nginx

&nbsp;       ports:

&nbsp;       - containerPort: 80

Apply it:

kubectl apply -f replicaset.yaml

2\. ReplicationController

ReplicationController is the older version of ReplicaSet. It also ensures that a specific number of pod replicas are running but lacks features like set-based selectors.



YAML for ReplicationController:

yaml

apiVersion: v1

kind: ReplicationController

metadata:

&nbsp; name: my-replicationcontroller

spec:

&nbsp; replicas: 3

&nbsp; selector:

&nbsp;   app: myapp

&nbsp; template:

&nbsp;   metadata:

&nbsp;     labels:

&nbsp;       app: myapp

&nbsp;   spec:

&nbsp;     containers:

&nbsp;     - name: mycontainer

&nbsp;       image: nginx

&nbsp;       ports:

&nbsp;       - containerPort: 80

Apply it:

kubectl apply -f replicationcontroller.yaml

🔧 3. Deployment

Deployment is the recommended way to manage replicas and rolling updates. It uses ReplicaSets internally.



YAML for Deployment:

yaml

apiVersion: apps/v1

kind: Deployment

metadata:

&nbsp; name: my-deployment

spec:

&nbsp; replicas: 3

&nbsp; selector:

&nbsp;   matchLabels:

&nbsp;     app: myapp

&nbsp; template:

&nbsp;   metadata:

&nbsp;     labels:

&nbsp;       app: myapp

&nbsp;   spec:

&nbsp;     containers:

&nbsp;     - name: mycontainer

&nbsp;       image: nginx

&nbsp;       ports:

&nbsp;       - containerPort: 80

Apply it:

kubectl apply -f deployment.yaml



**2.Kubernetes service types (ClusterIP, NodePort, LoadBalancer).**

In Kubernetes, ReplicaSet, ReplicationController, and Deployment are different ways to manage replication and scaling of Pods. Let’s go through each one, show how to deploy them, and compare their advantages and disadvantages.



1\. ReplicaSet

ReplicaSet ensures a specified number of pod replicas are running at all times.



YAML for ReplicaSet:

yaml

apiVersion: apps/v1

kind: ReplicaSet

metadata:

&nbsp; name: my-replicaset

spec:

&nbsp; replicas: 3

&nbsp; selector:

&nbsp;   matchLabels:

&nbsp;     app: myapp

&nbsp; template:

&nbsp;   metadata:

&nbsp;     labels:

&nbsp;       app: myapp

&nbsp;   spec:

&nbsp;     containers:

&nbsp;     - name: mycontainer

&nbsp;       image: nginx

&nbsp;       ports:

&nbsp;       - containerPort: 80

Apply it:

kubectl apply -f replicaset.yaml

2\. ReplicationController

ReplicationController is the older version of ReplicaSet. It also ensures that a specific number of pod replicas are running but lacks features like set-based selectors.



YAML for ReplicationController:

yaml

apiVersion: v1

kind: ReplicationController

metadata:

&nbsp; name: my-replicationcontroller

spec:

&nbsp; replicas: 3

&nbsp; selector:

&nbsp;   app: myapp

&nbsp; template:

&nbsp;   metadata:

&nbsp;     labels:

&nbsp;       app: myapp

&nbsp;   spec:

&nbsp;     containers:

&nbsp;     - name: mycontainer

&nbsp;       image: nginx

&nbsp;       ports:

&nbsp;       - containerPort: 80

Apply it:

kubectl apply -f replicationcontroller.yaml

3\. Deployment

Deployment is the recommended way to manage replicas and rolling updates. It uses ReplicaSets internally.



YAML for Deployment:

yaml

apiVersion: apps/v1

kind: Deployment

metadata:

&nbsp; name: my-deployment

spec:

&nbsp; replicas: 3

&nbsp; selector:

&nbsp;   matchLabels:

&nbsp;     app: myapp

&nbsp; template:

&nbsp;   metadata:

&nbsp;     labels:

&nbsp;       app: myapp

&nbsp;   spec:

&nbsp;     containers:

&nbsp;     - name: mycontainer

&nbsp;       image: nginx

&nbsp;       ports:

&nbsp;       - containerPort: 80

Apply it:

kubectl apply -f deployment.yaml.



**3.PersistentVolume (PV) and PersistentVolumeClaim (PVC).**

In Kubernetes, PersistentVolume (PV) and PersistentVolumeClaim (PVC) are used to manage persistent storage — storage that retains data across Pod restarts or rescheduling.



PersistentVolume:

A PV is a pre-provisioned storage unit in the cluster.



It’s an abstraction over physical storage (e.g., disk, NFS, cloud storage).



Created by the admin or dynamic provisioner.



Example PV YAML:

yaml

apiVersion: v1

kind: PersistentVolume

metadata:

&nbsp; name: my-pv

spec:

&nbsp; capacity:

&nbsp;   storage: 1Gi

&nbsp; accessModes:

&nbsp;   - ReadWriteOnce       # Pod can read/write from one node

&nbsp; persistentVolumeReclaimPolicy: Retain

&nbsp; hostPath:

&nbsp;   path: /mnt/data       # For demo (don't use in production)

2.PersistentVolumeClaim (PVC):

A PVC is a request for storage by a user.



It specifies size, access mode, and optionally storage class.



Kubernetes binds a PVC to a matching PV automatically.



Example PVC YAML:

yaml

apiVersion: v1

kind: PersistentVolumeClaim

metadata:

&nbsp; name: my-pvc

spec:

&nbsp; accessModes:

&nbsp;   - ReadWriteOnce

&nbsp; resources:

&nbsp;   requests:

&nbsp;     storage: 1Gi

How it works (Binding Process):

Admin/user creates a PV.



Developer creates a PVC.



Kubernetes matches PVC to PV based on:



storage size



accessModes



storageClassName (if specified)



PVC gets bound to a suitable PV.

Using PVC in a Pod

Once you have a PVC, you can mount it in a Pod.



Example Pod using a PVC:

yaml

apiVersion: v1

kind: Pod

metadata:

&nbsp; name: mypod

spec:

&nbsp; containers:

&nbsp; - name: mycontainer

&nbsp;   image: nginx

&nbsp;   volumeMounts:

&nbsp;   - mountPath: "/usr/share/nginx/html"

&nbsp;     name: my-storage

&nbsp; volumes:

&nbsp; - name: my-storage

&nbsp;   persistentVolumeClaim:

&nbsp;     claimName: my-pvc

Access Modes Explained

Mode	Description

ReadWriteOnce	One node can read/write

ReadOnlyMany	Multiple nodes can read

ReadWriteMany	Multiple nodes can read/write (NFS, CephFS)



Reclaim Policies

Policy	Description

Retain	PV is not deleted after PVC is deleted (manual cleanup required)

Delete	PV is deleted with PVC (for cloud volumes, etc.)

Recycle	(Deprecated) Wipes and makes PV available again



Static vs Dynamic Provisioning

Provisioning Type	Description

Static	Admin creates PV manually, users create PVC

Dynamic	Users create PVC with a StorageClass, and K8s automatically creates a PV



Example with Dynamic Provisioning (StorageClass + PVC):

yaml

\#StorageClass

apiVersion: storage.k8s.io/v1

kind: StorageClass

metadata:

&nbsp; name: standard

provisioner: kubernetes.io/aws-ebs  # or kubernetes.io/host-path for local



---



\# PVC

apiVersion: v1

kind: PersistentVolumeClaim

metadata:

&nbsp; name: dynamic-pvc

spec:

&nbsp; accessModes:

&nbsp;   - ReadWriteOnce

&nbsp; resources:

&nbsp;   requests:

&nbsp;     storage: 2Gi

&nbsp; storageClassName: standard

Summary

Component	Role

PV	Actual storage resource (pre-created or dynamic)

PVC	User request for storage (matched with PV)

Pod	Uses PVC as a volume



**4.Managing Kubernetes with Azure Kubernetes Service (AKS), Creating and managing AKS clusters, Scaling and upgrading AKS clusters.**

Managing Kubernetes using Azure Kubernetes Service (AKS) provides a managed Kubernetes environment with reduced operational overhead. Let’s break down how to create, scale, and upgrade AKS clusters.



AKS:

Azure Kubernetes Service (AKS) is a managed Kubernetes service provided by Microsoft Azure. AKS handles:



Cluster control plane (free)



Node provisioning, scaling, upgrades



Integrations with Azure Monitor, AAD, and more



1\. Creating an AKS Cluster

You can create an AKS cluster using:



Azure Portal (UI)



Azure CLI



Terraform or ARM templates



Azure CLI – Create Cluster

\# Set variables

RESOURCE\_GROUP=MyResourceGroup

CLUSTER\_NAME=MyAKSCluster



\# Create resource group

az group create --name $RESOURCE\_GROUP --location eastus



\# Create AKS cluster with default node pool

az aks create \\

&nbsp; --resource-group $RESOURCE\_GROUP \\

&nbsp; --name $CLUSTER\_NAME \\

&nbsp; --node-count 3 \\

&nbsp; --enable-addons monitoring \\

&nbsp; --generate-ssh-keys

Get credentials to access the cluster:

az aks get-credentials --resource-group $RESOURCE\_GROUP --name $CLUSTER\_NAME

Test the cluster:

kubectl get nodes

kubectl get pods -A

2\. Managing AKS Cluster (Common Operations)

Check Cluster Info:

az aks show --resource-group $RESOURCE\_GROUP --name $CLUSTER\_NAME

Connect to AKS with kubectl:

kubectl get services

kubectl get deployments

Enable/disable add-ons:

az aks enable-addons --addons monitoring --name $CLUSTER\_NAME --resource-group $RESOURCE\_GROUP

3\. Scaling AKS Clusters

You can scale manually or enable autoscaling.



Manual Scaling:

az aks scale --resource-group $RESOURCE\_GROUP --name $CLUSTER\_NAME --node-count 5

Enable Cluster Autoscaler:

az aks update \\

&nbsp; --resource-group $RESOURCE\_GROUP \\

&nbsp; --name $CLUSTER\_NAME \\

&nbsp; --enable-cluster-autoscaler \\

&nbsp; --min-count 2 \\

&nbsp; --max-count 5

AKS will now automatically add/remove nodes based on usage.



4\. Upgrading AKS Clusters

You can upgrade:



Control Plane



Node Pools



List available Kubernetes versions:

az aks get-upgrades --resource-group $RESOURCE\_GROUP --name $CLUSTER\_NAME

Upgrade cluster (control plane + nodes):

az aks upgrade \\

&nbsp; --resource-group $RESOURCE\_GROUP \\

&nbsp; --name $CLUSTER\_NAME \\

&nbsp; --kubernetes-version <new-version>

Upgrade node pool only:

az aks nodepool upgrade \\

&nbsp; --resource-group $RESOURCE\_GROUP \\

&nbsp; --cluster-name $CLUSTER\_NAME \\

&nbsp; --name nodepool1 \\

&nbsp; --kubernetes-version <new-version>

Monitoring and Logs

Use Azure Monitor for Containers



Enable it via --enable-addons monitoring



View logs in Log Analytics Workspace



Secure AKS

Use Azure Active Directory (AAD) integration for RBAC



Use Network Policies



Use Azure Key Vault for secrets



Summary Table

Operation	Command

Create AKS Cluster	az aks create

Get Credentials	az aks get-credentials

Scale Cluster	az aks scale

Enable Autoscaler	az aks update --enable-cluster-autoscaler

Upgrade Cluster	az aks upgrade

View Cluster Info	az aks show.



**5.Configure liveliness and readiness probes for pods in AKS cluster.**

Configuring liveness and readiness probes in an AKS (Azure Kubernetes Service) cluster works exactly the same as in any standard Kubernetes cluster. These probes are essential for managing application health checks.



Probe Type	Purpose

Liveness Probe	Checks if the app is alive. If it fails, Kubernetes restarts the pod.

Readiness Probe	Checks if the app is ready to serve traffic. If it fails, the pod is removed from the service endpoint.



Types of Probes

HTTP probe: Sends an HTTP GET request



TCP probe: Tries to open a TCP socket



Exec probe: Runs a command inside the container



Example: Configure Probes in a Pod

Here’s an example of an NGINX deployment with both liveness and readiness probes:



nginx-probes.yaml

yaml

apiVersion: apps/v1

kind: Deployment

metadata:

&nbsp; name: nginx-probe-demo

spec:

&nbsp; replicas: 2

&nbsp; selector:

&nbsp;   matchLabels:

&nbsp;     app: nginx

&nbsp; template:

&nbsp;   metadata:

&nbsp;     labels:

&nbsp;       app: nginx

&nbsp;   spec:

&nbsp;     containers:

&nbsp;     - name: nginx

&nbsp;       image: nginx:1.25

&nbsp;       ports:

&nbsp;       - containerPort: 80

&nbsp;       livenessProbe:

&nbsp;         httpGet:

&nbsp;           path: /

&nbsp;           port: 80

&nbsp;         initialDelaySeconds: 10

&nbsp;         periodSeconds: 5

&nbsp;         timeoutSeconds: 2

&nbsp;         failureThreshold: 3

&nbsp;       readinessProbe:

&nbsp;         httpGet:

&nbsp;           path: /

&nbsp;           port: 80

&nbsp;         initialDelaySeconds: 5

&nbsp;         periodSeconds: 5

&nbsp;         timeoutSeconds: 2

&nbsp;         successThreshold: 1

Steps to Deploy on AKS

1\. Connect to AKS cluster

az aks get-credentials --resource-group <YourResourceGroup> --name <YourAKSCluster>

2\. Apply the manifest

bash

Copy

Edit

kubectl apply -f nginx-probes.yaml

3\. Check probe status

bash

Copy

Edit

kubectl describe pod -l app=nginx

kubectl get pods

Test Liveness Failure (Optional Demo)

You can simulate liveness probe failure by creating a container that crashes if /healthz endpoint is down:



Example:

yaml

livenessProbe:

&nbsp; httpGet:

&nbsp;   path: /healthz

&nbsp;   port: 8080

&nbsp; initialDelaySeconds: 5

&nbsp; periodSeconds: 5

If your app returns non-200 on /healthz, the pod will be restarted.



Probe Parameters Explained

Field	Description

initialDelaySeconds	Time to wait after container starts before probing

periodSeconds	How often to probe

timeoutSeconds	Probe timeout

successThreshold	Number of successes before marking pod as ready

failureThreshold	Number of failures before taking action (restart or mark not-ready)



Summary

Liveness probes: Ensure your app doesn’t get stuck.



Readiness probes: Ensure traffic is only sent to ready pods.



Probes help AKS (or any Kubernetes cluster) maintain a healthy, resilient system.



**6.Configure Taints and Tolerants.**

In Kubernetes (including AKS), Taints and Tolerations work together to control which pods can be scheduled on which nodes.



Taints and Tolerations are:

Concept	Description

Taint (Node-level)	Prevents pods from being scheduled on a node unless the pod tolerates the taint.

Toleration (Pod-level)	Allows the pod to tolerate a taint and be scheduled on the tainted node.



1\. Add Taint to a Node

Syntax:

kubectl taint nodes <node-name> key=value:effect

key=value is a label.



effect can be:



NoSchedule: Prevent scheduling



PreferNoSchedule: Try to avoid scheduling



NoExecute: Evict existing pods if they don’t tolerate



Example:

kubectl taint nodes aks-nodepool1-12345678 dedicated=backend:NoSchedule

This taint says: "Only pods that tolerate dedicated=backend:NoSchedule can run here."



2\. Add Toleration to a Pod (or Deployment)

Example YAML (tolerating the above taint):

yaml

apiVersion: v1

kind: Pod

metadata:

&nbsp; name: backend-pod

spec:

&nbsp; containers:

&nbsp; - name: backend

&nbsp;   image: nginx

&nbsp; tolerations:

&nbsp; - key: "dedicated"

&nbsp;   operator: "Equal"

&nbsp;   value: "backend"

&nbsp;   effect: "NoSchedule"

In a Deployment:

yaml

apiVersion: apps/v1

kind: Deployment

metadata:

&nbsp; name: backend-deploy

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

&nbsp;       image: nginx

&nbsp;     tolerations:

&nbsp;     - key: "dedicated"

&nbsp;       operator: "Equal"

&nbsp;       value: "backend"

&nbsp;       effect: "NoSchedule"

3\. Remove a Taint

kubectl taint nodes aks-nodepool1-12345678 dedicated=backend:NoSchedule-

Note the - at the end — it removes the taint.



Use Cases for Taints \& Tolerations

Use Case	Taint/Toleration

Separate workloads (frontend/backend)	Taint backend nodes

Reserve nodes for system pods	 Taint with CriticalAddonsOnly

Schedule only GPU workloads on GPU nodes	 Taint GPU nodes

Prevent certain pods from landing on specific nodes	 Use taints and not tolerate them



Effects of Taints

Effect	Behavior

NoSchedule	Pods that don't tolerate this won’t be scheduled

PreferNoSchedule	Scheduler avoids, but may schedule

NoExecute	Evicts non-tolerating pods and prevents new ones



Summary

Component	Purpose

Taint	Applied to node to repel pods

Toleration	Applied to pod to match and bypass taints

Command	kubectl taint nodes ...

Field in YAML	tolerations under spec.



**7.Create and attach persistent volume claims to pods.**

To create and attach Persistent Volume Claims (PVCs) to Pods in Kubernetes (or AKS), you need to follow a 3-step process:



Step-by-Step Guide

1\. Create a Persistent Volume (PV)

(or use dynamic provisioning with StorageClass)



2\. Create a Persistent Volume Claim (PVC)

3\. Attach PVC to a Pod

Example: Static Provisioning

Step 1: Create a PersistentVolume (PV)

yaml

\# pv.yaml

apiVersion: v1

kind: PersistentVolume

metadata:

&nbsp; name: my-pv

spec:

&nbsp; capacity:

&nbsp;   storage: 1Gi

&nbsp; accessModes:

&nbsp;   - ReadWriteOnce

&nbsp; persistentVolumeReclaimPolicy: Retain

&nbsp; hostPath:  # Use only for local testing (not recommended in production/AKS)

&nbsp;   path: "/mnt/data"

Apply it:

kubectl apply -f pv.yaml

Step 2: Create a PersistentVolumeClaim (PVC)

yaml

\# pvc.yaml

apiVersion: v1

kind: PersistentVolumeClaim

metadata:

&nbsp; name: my-pvc

spec:

&nbsp; accessModes:

&nbsp;   - ReadWriteOnce

&nbsp; resources:

&nbsp;   requests:

&nbsp;     storage: 1Gi

Apply it:

kubectl apply -f pvc.yaml

Step 3: Create a Pod and Mount the PVC

yaml

\# pod.yaml

apiVersion: v1

kind: Pod

metadata:

&nbsp; name: my-pod

spec:

&nbsp; containers:

&nbsp; - name: mycontainer

&nbsp;   image: nginx

&nbsp;   volumeMounts:

&nbsp;   - mountPath: "/usr/share/nginx/html"

&nbsp;     name: my-storage

&nbsp; volumes:

&nbsp; - name: my-storage

&nbsp;   persistentVolumeClaim:

&nbsp;     claimName: my-pvc

Apply it:

kubectl apply -f pod.yaml

Dynamic Provisioning (Recommended for AKS)

Use a StorageClass (e.g., Azure Disk or Azure File) and only define a PVC.



yaml

\# dynamic-pvc.yaml

apiVersion: v1

kind: PersistentVolumeClaim

metadata:

&nbsp; name: azure-managed-disk-pvc

spec:

&nbsp; accessModes:

&nbsp;   - ReadWriteOnce

&nbsp; storageClassName: managed-premium

&nbsp; resources:

&nbsp;   requests:

&nbsp;     storage: 5Gi

AKS automatically provisions Azure Disk using managed-premium storage class.



Pod using dynamically provisioned PVC

yaml

apiVersion: v1

kind: Pod

metadata:

&nbsp; name: azure-disk-pod

spec:

&nbsp; containers:

&nbsp; - name: app

&nbsp;   image: nginx

&nbsp;   volumeMounts:

&nbsp;   - mountPath: "/mnt/azure"

&nbsp;     name: azure-storage

&nbsp; volumes:

&nbsp; - name: azure-storage

&nbsp;   persistentVolumeClaim:

&nbsp;     claimName: azure-managed-disk-pvc

Verify

kubectl get pv

kubectl get pvc

kubectl describe pvc my-pvc

kubectl describe pod my-pod

Summary

Step	What You Do	File

1	Define a Persistent Volume (optional with dynamic provisioning)	pv.yaml

2	Create a Persistent Volume Claim (PVC)	pvc.yaml

3	Attach PVC in Pod/Deployment	pod.yaml.



**8.Configure health probes for pods.**

Configuring health probes (liveness and readiness) in Kubernetes pods is essential for maintaining a resilient and self-healing application.



These probes tell Kubernetes:



Liveness Probe: Is the app alive? If not, restart the container.



Readiness Probe: Is the app ready to receive traffic? If not, temporarily remove it from the service endpoints.



Types of Probes

Type	Use Case

httpGet	App exposes an HTTP health endpoint (e.g., /health)

tcpSocket	App listens on a TCP port (e.g., database)

exec	Run a command inside the container



Example: Pod with Liveness and Readiness Probes

Container Image: nginx

apiVersion: v1

kind: Pod

metadata:

&nbsp; name: nginx-probe-pod

spec:

&nbsp; containers:

&nbsp; - name: nginx

&nbsp;   image: nginx:1.25

&nbsp;   ports:

&nbsp;   - containerPort: 80

&nbsp;   livenessProbe:

&nbsp;     httpGet:

&nbsp;       path: /

&nbsp;       port: 80

&nbsp;     initialDelaySeconds: 10

&nbsp;     periodSeconds: 5

&nbsp;     failureThreshold: 3

&nbsp;   readinessProbe:

&nbsp;     httpGet:

&nbsp;       path: /

&nbsp;       port: 80

&nbsp;     initialDelaySeconds: 5

&nbsp;     periodSeconds: 5

&nbsp;     failureThreshold: 2

Probe Parameters Explained

Field	Description

initialDelaySeconds	Time to wait after container starts before first probe

periodSeconds	Frequency of probing

timeoutSeconds	Time to wait before probe times out

successThreshold	Minimum successes to be considered healthy (for readiness)

failureThreshold	Failures before considering container as failed



Other Probe Types

1\. TCP Socket Probe

livenessProbe:

&nbsp; tcpSocket:

&nbsp;   port: 3306

&nbsp; initialDelaySeconds: 5

&nbsp; periodSeconds: 10

2\. Exec Probe

readinessProbe:

&nbsp; exec:

&nbsp;   command:

&nbsp;   - cat

&nbsp;   - /tmp/ready

&nbsp; initialDelaySeconds: 5

&nbsp; periodSeconds: 10

Test the Configuration

After applying the YAML:



kubectl apply -f nginx-probe-pod.yaml

kubectl describe pod nginx-probe-pod

kubectl get pods

If the probe fails:



Liveness: container is restarted



Readiness: removed from service endpoints (no traffic sent)



Apply in AKS

If you are working with AKS, the process is the same:



Connect to cluster:



az aks get-credentials --resource-group my-rg --name my-aks-cluster

Apply:



kubectl apply -f nginx-probe-pod.yaml

Summary

Probe	Use	Action if fails

Liveness Probe	App is running correctly	Pod will be restarted

Readiness Probe	App is ready to accept traffic	Removed from service load.



**9.Configure autoscaling in your cluster (Horizontal scaling).**

To configure horizontal pod autoscaling (HPA) in your Kubernetes or AKS cluster, you'll use the HorizontalPodAutoscaler resource. HPA automatically scales the number of pod replicas based on CPU/memory utilization or custom metrics.



Prerequisites

Metrics Server must be running in the cluster:



In AKS, it’s installed by default.



For other clusters (like Minikube), install it manually:



kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

Your deployment should request CPU/memory resources:



HPA only works if resources.requests are set.



Step-by-Step: Horizontal Pod Autoscaler

1\. Create a Deployment with CPU requests

\# nginx-deployment.yaml

apiVersion: apps/v1

kind: Deployment

metadata:

&nbsp; name: nginx-hpa-demo

spec:

&nbsp; replicas: 1

&nbsp; selector:

&nbsp;   matchLabels:

&nbsp;     app: nginx-hpa

&nbsp; template:

&nbsp;   metadata:

&nbsp;     labels:

&nbsp;       app: nginx-hpa

&nbsp;   spec:

&nbsp;     containers:

&nbsp;     - name: nginx

&nbsp;       image: nginx

&nbsp;       resources:

&nbsp;         requests:

&nbsp;           cpu: "100m"

&nbsp;           memory: "200Mi"

&nbsp;         limits:

&nbsp;           cpu: "200m"

&nbsp;           memory: "300Mi"

&nbsp;       ports:

&nbsp;       - containerPort: 80

kubectl apply -f nginx-deployment.yaml

2\. Create Horizontal Pod Autoscaler

kubectl autoscale deployment nginx-hpa-demo \\

&nbsp; --cpu-percent=50 \\

&nbsp; --min=1 \\

&nbsp; --max=5

This sets up autoscaling based on average CPU usage across pods.



3\. View the HPA Status

kubectl get hpa

Output example:

NAME               REFERENCE                     TARGETS   MINPODS   MAXPODS   REPLICAS   AGE

nginx-hpa-demo     Deployment/nginx-hpa-demo     30%/50%   1         5         1          2m

4\. Generate Load to Trigger Autoscaling (Optional Test)

Use a busybox pod to send requests:

kubectl run -i --tty load-generator --image=busybox /bin/sh



\# Inside the busybox shell:

while true; do wget -q -O- http://nginx-hpa-demo.default.svc.cluster.local; done

Wait 1–2 minutes, then run:

kubectl get hpa

kubectl get pods

You should see replicas scaling up.



5\. Clean Up (Optional)

kubectl delete hpa nginx-hpa-demo

kubectl delete deployment nginx-hpa-demo

kubectl delete pod load-generator

Summary

Component	Purpose

metrics-server	Collects resource usage

Deployment	Application workload with resource requests

HorizontalPodAutoscaler	Scales replicas based on CPU/memory/custom metrics.



























