### KubernetesLab  

# Containerizarea

Containerizarea presupune plasarea unei componente software, a mediului, dependintelor si configuratiei sale, într-o unitate izolata numità container. Acest lucru face posibila implementarea unei aplicatii in mod consevent in orice mediu, fie on-premise sau bazat pe cloud.

# Docker
Instrument de containerizare care impacheteaza aplicatia si toate dependintele impreunã. Astfel, asa ne asiguram ca aplicatia noastra merge bine in orice mediu.

DockerFile
```
FROM node:14-alpine3.16 

WORKDIR /app 

COPY . .

RUN npm install

CMD [ "npm", "start"]

```
- FROM node:14-alpine3.16 : Imaginea de bază folosită pentru construirea containerului, în acest caz, o versiune specifică de Node.js
- WORKDIR /app            : Setează directorul de lucru în container la /app. Orice comandă ulterioară va fi executată în acest director.
- COPY . .                : Copiază toate fișierele din directorul curent al host-ului (unde se execută Dockerfile) în directorul de lucru actual al containerului (/app).
- RUN npm install         : Execută comanda npm install în directorul de lucru al containerului. Aceasta va instala toate dependențele specificate în fișierul package.json, care trebuie să                             fie printre fișierele copiate în container.
- CMD [ "npm", "start"]   : Specifică comanda implicită care va fi executată când containerul este pornit. În acest caz, pornește aplicația Node.js folosind scriptul definit în package.json sub comanda start.

## Importanța pentru Kubernetes

* Izolare
* Portabilitate
* Scalabilitate și Management

# Introducere in Kubernetes

Kubernetes (prescurtat K8s) este cel mai popular orchestrator, folosit la scara largă și oferit ca și CaaS (Container as a Service) de toți vendorii de infrastructură (Amazon, Google, Microsoft). Este considerat standard în materie de deployment al serviciilor orchestrate.
Kubernetes simplifică sarcinile complexe de gestionare a aplicațiilor containerizate, oferind mecanisme robuste pentru implementare, întreținere și scalare.


## **Arhitectura Kubernetes**
Kubernetes este un sistem format din mai multe [Componente](https://kubernetes.io/docs/concepts/overview/components/#master-components)

1. Kubectl - CLI pentru configurarea clusterului și managementului aplicațiilor. Asemănător cu comanda docker
2. Node - nodul fizic din cadrul unui cluster
3. Kubelet - agentul (daemon) de Kubernetes care rulează pe fiecare nod
4. Control Plane - colecția de noduri care fac management la cluster. Include API Server, Scheduler, Controller Manager, CoreDNS, Etcd.

## Instalare Kubernetes
Cea mai buna metodă de a învăța este de a seta  local un cluster cu un singur nod de Kubernetes. Acest lucru se poate face în mai multe moduri:

- [Minikube](https://kubernetes.io/docs/concepts/overview/components/#master-components](https://minikube.sigs.k8s.io/docs/start/))
- [MicroK8s](https://microk8s.io)

## Crearea si rularea unui Pod Kubernetes
Putem crea și rula pod-uri în două maniere: imperativă (prin comenzi cu parametri) și declarativă (folosind fișiere de configurare).

Dacă dorim să rulam un pod in mod declarativ, folosim următoarea comanda:`kubectl run <pod-name> –-image <image name>`.
Pentru a testa daca o comanda ar fi executata cu success dar nu vrem sa cream resursa Kubernetes putem sa adaugam la final `--dry-run=client`.
Mai multe informatii si exemple de folosire [aici](https://kubernetes.io/docs/concepts/workloads/pods/).

Dacă dorim să afișăm toate pod-urile folosim comanda `kubectl get pods` și dacă dorim să listăm toate namespace-urile folosim `kubectl get namespaces`. De asemenea, având în vedere că un pod poate rula în cadrul unui namespace, putem să afișăm toate pod-urile din cadrul unui namespace: `kubectl get pods -n <namespace>`
Daca dorim sa vedem informatii cu privire la pod-uri putem sa rulam comenzi precum:
```
kubectl exec -it mypod ls                     # Pentru a rula o comanda in interiorul unui pod (mypod trebuie sa existe deja in cluster) 
kubectl logs <nume-pod>                       # afiseaza loguri
kubectl delete pods/<nume-pod>                # sterge un pod

```

Un pod poate rula în cadrul unui namespace. Dacă dorim acest lucru, creăm mai întâi un namespace folosind comanda `kubectl create namespace <namespace-name>`. 

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80

```
Pentru a crea acest pod, folosim comanda apply și specificăm fișierul: `kubectl apply -f nginx.yaml`.

### Obiecte in Kubernetes 
Pentru a intelege cum obiectele Kubernetes sunt reprezentate in format .yaml se poate accessa [aici](https://kubernetes.io/docs/concepts/overview/working-with-objects/)

# Deployments in Kubernetes
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 # Spune Deployment-ului să ruleze 2 pod-uri care corespund template-ului
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```
# ConfigMaps și Secrets
Un ConfigMap este un obiect folosit pentru a stoca într-un format de tipul cheie-valoare date care nu sunt sensitive. Un pod poate consuma un ConfigMap ca o variabilă de mediu, ca un argument în linie de comandă sau ca un fișier de configurare într-un volum. Un astfel de obiect oferă opțiunea de a decupla configurația specifica unui mediu de imaginile de container și de codul aplicației, ceea ce sporește portabilitatea aplicațiilor.

```
apiVersion: v1
kind: ConfigMap
metadata:
    name: game-demo
data:
    # property-like keys; each key maps to a simple value
    player_initial_lives: "3"
```

- Vom utiliza comanda `docker build -t numeImagine:TAG . ` pentru a construi imaginea aferenta aplicatiei.
- Vom publica imaginea prin `docker push numeImagine:TAG `
- O sa inlocuim in deployment `image: numeImagine:TAG`
mai multe despre partea de Imagine si Docker [aici]()


Pentru Deployment-ul aplicatiei o sa adaugam un obiect Kubernetes de tip Service, care configurează modul în care traficul extern este direcționat către unul sau mai multe Pods rulând în cluster.
```
apiVersion: v1
kind: Service
metadata:  
  name: my-nodeport-service
spec:
  selector:    
    app: nginx
  type: NodePort
  ports:  
  - name: http
    port: 80
    targetPort: 80
    nodePort: 30036
    protocol: TCP

```
Tipul de Service. `NodePort` este o configurație care permite accesul extern la Pods din cluster prin deschiderea unui port specific (nodePort) pe toate nodurile (mașinile fizice sau VM-urile) din cluster.

Vom rula urmatoarele comenzi pentru a verifica procedura:
```
kubectl get pods # sa vedem instantele aplicatiei
kubectl get svc  # sa vedem service ul creat

```




# Homework1

- 

app.js
```
const express = require('express')
const app = express()
const port = 3000

app.get('/', (req, res) => {
  res.send('Hello World!')
})

app.listen(port, () => {
  console.log(`Example app listening on port ${port}`)
})
```
### Cerinte:
- Pe baza app.js creati o imagine Dockerfile.
- Porniti un container pe baza imaginii create
- Creati un Deployment

## Optional
- Creati un pod separat pentru a rula o baza de date
- Conectati-va la pod-ul cu baza de date si incercati sa rulati comenzi din terminal.
  


# TerraformEksKarpenter


# EKS Cluster with Karpenter Autoscaler - Terraform Module

## Overview
This repository provides Terraform scripts to set up an **EKS cluster** with **Karpenter Autoscaler** for efficient scaling and cost optimization. It includes:
- **EKS Cluster** provisioning with associated IAM roles and policies.
- **Karpenter Integration** for dynamic node scaling.
- Support for x86 and ARM-based (Graviton) workloads using Kubernetes `nodeSelector`.

## Prerequisites

### Tools Required
- [Terraform](https://www.terraform.io/downloads.html) (v1.3.0 or later)
- [AWS CLI](https://aws.amazon.com/cli/) (configured with necessary credentials)
- [kubectl](https://kubernetes.io/docs/tasks/tools/) (for cluster interaction)
- [Helm](https://helm.sh/) (for deploying Karpenter)

### Step 1: Clone the Repository
```bash
git clone <repo-url>
cd <repo-directory>
```

### Step 2: Update the Main Terraform Configuration
Ensure the main Terraform configuration file correctly references the modules and variables. Here's an example:
```hcl
module "vpc" {
  vpc_name                 = "eks-vpc"
  source                   = "./modules/vpc"
  vpc_cidr_block           = "192.168.0.0/16"
  public_cidrs             = ["192.168.64.0/19", "192.168.96.0/19"]
  private_cidrs            = ["192.168.128.0/19", "192.168.32.0/19"]
  availability_zones       = ["us-east-1a", "us-east-1b"]
  nat_name                 = "eks-nat"
  igw_name                 = "eks-igw"
  public_route_table_name  = "eks-public-route"
  private_route_table_name = "eks-private-route"
  cluster_name   = "demo"
```

## Steps to Deploy the EKS Cluster with Karpenter Autoscaler

### Step 3: Initialize, Plan and Apply Terraform
Run the following command to initialize the Terraform project:
```bash
terraform init
```
```bash
terraform plan
```
```bash
terraform apply
```
You will see something like this : 

<img width="678" alt="terraformApply" src="https://github.com/user-attachments/assets/b8d1579c-22b4-4d08-bde7-8a0ef1bea0de" />

In order to use the kubernetes cluster that you just created above you need to:

```bash
aws eks update-kubeconfig --name demo --region us-east-1

Updated context arn:aws:eks:region:xxxxxxxxx12414:cluster/name-of-the-cluster in /Users/user/.kube/config
``` 

### Step 4: Apply Kubernetes Manifests

First you have to 
```bash
$ cd k8s/
$ kubectl apply -f Provisioner/yaml
$ kubectl apply -f armtest.yaml
$ kubectl apply -f amd.yaml
```

1. Provisioner.yaml ( contains Provisioner needed for karpenter and the AWSNodeTemplate)
```yaml
apiVersion: karpenter.sh/v1alpha5
kind: Provisioner
metadata:
  name: default
spec:
  ttlSecondsAfterEmpty: 60 
  ttlSecondsUntilExpired: 604800 
  limits:
    resources:
      cpu: 100 
  requirements:
    - key: karpenter.k8s.aws/instance-family
      operator: In
      values: [c5, m5, c6g]
    - key: karpenter.k8s.aws/instance-size
      operator: NotIn
      values: [nano, micro ]
    - key: kubernetes.io/arch
      operator: In
      values: [ amd64, arm64 ] 
    - key: karpenter.sh/capacity-type
      operator: In
      values: ["on-demand", "spot"]
  providerRef:
    name: my-provider
---
apiVersion: karpenter.k8s.aws/v1alpha1
kind: AWSNodeTemplate
metadata:
  name: my-provider
spec:
  subnetSelector:
    kubernetes.io/cluster/demo: owned
  securityGroupSelector:
    kubernetes.io/cluster/demo: owned
  instanceProfile: KarpenterNodeInstanceProfile
```
2. armtest.yaml (to provision Graviton instance on the arm64 Architecture)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-arm64
  annotations:
    karpenter.sh/provisioner-name: default
spec:
  nodeSelector:
    kubernetes.io/arch: arm64
  containers:
  - name: nginx
    image: nginx
    resources:
      requests:
        cpu: "1"
        memory: "1Gi"
```
3. amd.yaml (to provision x86 instance)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod-amd64
spec:
  nodeSelector:
    kubernetes.io/arch: amd64
  containers:
  - name: test
    image: nginx
```

###After everything is applied 

to show the status of your pods:
```bash
kubectl get pods
kubectl describe pod <pod-name>
kubectl logs <pod-name> 

to see the activity of the provisioner
kubectl logs -n karpenter -l app.kubernetes.io/name=karpenter
```
<img width="602" alt="amd64Node" src="https://github.com/user-attachments/assets/7b5a9e73-6f3c-4ad8-8922-2a99570e93d5" />

<img width="878" alt="arm64Node" src="https://github.com/user-attachments/assets/839e2e25-3fad-4645-9b3c-241642073b16" />


## 🚀 Optimizing GPU Costs on EKS with GPU Slicing and Karpenter Autoscaler

---

For machine learning, utilizing GPUs powers computational workloads with unmatched efficiency. However, the trend is shifting toward cloud computing instead of relying on individual bare-metal machines. Why? With the cloud, you only pay for what you use, eliminating the need to invest in expensive hardware that might sit idle for extended periods.

**How can we optimize GPU utilization in the cloud?** 🤔

When using virtual machines (VMs), you pay for the entire device, including GPUs, even when not in use 24/7. Kubernetes, on the other hand, introduces elastic node scaling, making it inherently more cloud-native and cost-efficient. By leveraging **Karpenter**, a node autoscaler for Kubernetes running on Amazon EKS, you can dynamically provision nodes based on real-time workload demands. 

Additionally, to maximize GPU utilization, you'll need **NVIDIA's Kubernetes plugin**, a DaemonSet that automatically:

- 📊 Exposes the number of GPUs on each node in your cluster.
- ✅ Monitors the health of GPUs for proactive resource management.
- 🖥️ Enables GPU-powered containers within your Kubernetes cluster.
- ⏱️ Supports GPU time-slicing, allowing multiple pods to share GPUs, significantly reducing costs.

### **The Key Advantages of GPU Utilization with Karpenter** 🌟

Karpenter not only offers cost-saving benefits by auto-scaling GPU nodes but also provides a flexible way to allocate GPU resources across various workloads. Imagine owning tens of applications requiring GPUs at different times. With Karpenter, you can schedule resources more effectively, ensuring efficiency and avoiding idle GPUs.


### GPU Slicing Techniques 🛠️

### 1. **Virtual GPU (vGPU)**

- **What is it?** Share a physical GPU among multiple virtual machines (VMs) with dedicated GPU portions.
- **✅ Best for**: Virtual desktops, cloud gaming, and containerized AI/ML workloads.
- **✨ Benefits**:
  - Predictable performance per VM.
  - Strong security and isolation.
  - Up to 20 partitions per GPU (e.g., NVIDIA A100 80GB with vCS).

---

### 2. **Multi-Instance GPU (MIG)**

- **What is it?** Partition a GPU into isolated instances at the hardware level.
- **✅ Best for**: High-performance AI model training, inference servers, and HPC workloads.
- **✨ Benefits**:
  - Superior performance with minimal overhead.
  - Dedicated compute, memory, and bandwidth for each instance.
  - Isolation prevents interference between tasks.

---

### 3. **GPU Time-Slicing**

- **What is it?** Share GPU resources by slicing processing time into intervals for different tasks.
- **✅ Best for**: Background tasks, batch processing, or non-critical workloads.
- **✨ Benefits**:
  - Maximizes GPU utilization with no need for specialized hardware.
  - Reduces operational costs by sharing GPU access.

---

## Practical Solution: GPU Slicing with EKS and Karpenter 🔧

---

### Arhitecture 
![Architecture](https://github.com/user-attachments/assets/c89ad654-d7ee-4fd4-bf2f-fab1cd37d731)

It’s pretty straightforward: the application will choose a karpenter provisioner with a selector. The karpenter provisioner will create nodes based on the launch template in that provisioner.

We already have eks with karpenter from the terraform above so no need to install again 

### Apply Kubernetes manifests 

#### Just like we had for the Graviton and x86 a Karpenter provisioner , we need one in this case as well in order for GPU workloads to dynamically scale nodes.

#### GPU-Dedicated Provisioner

```yaml
apiVersion: karpenter.sh/v1alpha5
kind: Provisioner
metadata:
  name: gpu
spec:
  ttlSecondsAfterEmpty: 60 # Scale down nodes after 60 seconds without workloads
  ttlSecondsUntilExpired: 604800 # Expire nodes after 7 days
  labels:
    jina.ai/node-type: gpu
    jina.ai/gpu-type: nvidia
  requirements:
    # Match existing subnet and security group selectors
    - key: karpenter.k8s.aws/instance-family
      operator: In
      values: [g4dn]
    - key: karpenter.sh/capacity-type
      operator: In
      values: ["spot", "on-demand"]
    - key: kubernetes.io/arch
      operator: In
      values: [amd64]
  taints:
    - key: nvidia.com/gpu
      effect: "NoSchedule"
  provider:
    instanceProfile: KarpenterInstanceProfile 
    subnetSelector:
      kubernetes.io/cluster/demo: owned 
    securityGroupSelector:
      kubernetes.io/cluster/demo: owned 
    tags:
      karpenter.sh/discovery: demo 

```

#### GPU-Shared Provisioner
```yaml
apiVersion: karpenter.sh/v1alpha5
kind: Provisioner
metadata:
  name: gpu-shared
spec:
  ttlSecondsAfterEmpty: 60
  ttlSecondsUntilExpired: 604800
  labels:
    jina.ai/node-type: gpu-shared
    jina.ai/gpu-type: nvidia
    nvidia.com/device-plugin.config: shared_gpu
  requirements:
    - key: karpenter.k8s.aws/instance-family
      operator: In
      values: [g4dn]
    - key: karpenter.sh/capacity-type
      operator: In
      values: ["spot", "on-demand"]
    - key: kubernetes.io/arch
      operator: In
      values: [amd64]
  taints:
    - key: nvidia.com/gpu-shared
      effect: "NoSchedule"
  provider:
    instanceProfile: KarpenterInstanceProfile 
    subnetSelector:
      kubernetes.io/cluster/demo: owned 
    securityGroupSelector:
      kubernetes.io/cluster/demo: owned 
    tags:
      karpenter.sh/discovery: demo 

```

#### NVIDIA DaemonSet with Shared GPU Configuration

```yaml
config:
  map:
    default: |-
      version: v1
      flags:
        migStrategy: "none"
        failOnInitError: true
        nvidiaDriverRoot: "/"
        plugin:
          passDeviceSpecs: false
          deviceListStrategy: envvar
          deviceIDStrategy: uuid
    shared_gpu: |-
      version: v1
      flags:
        migStrategy: "none"
        failOnInitError: true
        nvidiaDriverRoot: "/"
        plugin:
          passDeviceSpecs: false
          deviceListStrategy: envvar
          deviceIDStrategy: uuid
      sharing:
        timeSlicing:
          renameByDefault: false
          resources:
            - name: nvidia.com/gpu
              replicas: 10
nodeSelector:
  jina.ai/gpu-type: nvidia
```

https://gist.github.com/tarrantro/6c6bfa1e114e4ca4762a5facaeb0c355

The Provisioners above is set to use corelated launchtemplates to provision GPU nodes with labels and taints.
Launch template (only GPU)
Secondly, we need to deploy the NVIDIA k8s plugin with time-slicing config and default config and set up a node selector so the daemonset will only run on the GPU instances.

### nvdp.yaml
```yaml
config:
  # ConfigMap name if pulling from an external ConfigMap
  name: ""
  # Set of named configs to build an integrated ConfigMap from
  map: 
    default: |-
      version: v1
      flags:
        migStrategy: "none"
        failOnInitError: true
        nvidiaDriverRoot: "/"
        plugin:
          passDeviceSpecs: false
          deviceListStrategy: envvar
          deviceIDStrategy: uuid
    shared_gpu: |-
      version: v1
      flags:
        migStrategy: "none"
        failOnInitError: true
        nvidiaDriverRoot: "/"
        plugin:
          passDeviceSpecs: false
          deviceListStrategy: envvar
          deviceIDStrategy: uuid
      sharing:
        timeSlicing:
          renameByDefault: false
          resources:
          - name: nvidia.com/gpu
            replicas: 10
nodeSelector: 
  jina.ai/gpu-type: nvidia
```
### Install NVIDIA Kubernetes Plugin

The NVIDIA Kubernetes Device Plugin is essential for enabling GPU sharing and monitoring. Install it using Helm :

```bash
helm repo add nvdp https://nvidia.github.io/k8s-device-plugin
helm repo update
helm upgrade -i nvdp nvdp/nvidia-device-plugin \
  --namespace nvidia-device-plugin \
  --create-namespace
```
# Deploy GPU ( Dedicated Nodes) 
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gpu-dedicated
  labels:
    app: gpu-dedicated
spec:
  replicas: 1
  selector:
    matchLabels:
      app: gpu-dedicated
  template:
    metadata:
      labels:
        app: gpu-dedicated
    spec:
      nodeSelector:
        jina.ai/node-type: gpu 
        karpenter.sh/provisioner-name: gpu 
      tolerations:
        - key: nvidia.com/gpu
          operator: Exists
          effect: NoSchedule
      containers:
        - name: gpu-container
          image: tensorflow/tensorflow:latest-gpu
          imagePullPolicy: Always
          command: ["python"]
          args: ["-u", "-c", "import tensorflow as tf; print(tf.test.gpu_device_name())"]
          resources:
            limits:
              nvidia.com/gpu: 1 

```

# Deploy GPU-Shared Deployment ( Time-Slicing Nodes) 
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gpu-shared
  labels:
    app: gpu-shared
spec:
  replicas: 1
  selector:
    matchLabels:
      app: gpu-shared
  template:
    metadata:
      labels:
        app: gpu-shared
    spec:
      nodeSelector:
        jina.ai/node-type: gpu-shared 
        karpenter.sh/provisioner-name: gpu-shared 
      tolerations:
        - key: nvidia.com/gpu-shared
          operator: Exists
          effect: NoSchedule
      containers:
        - name: gpu-shared-container
          image: tensorflow/tensorflow:latest-gpu
          imagePullPolicy: Always
          command: ["python"]
          args: ["-u", "-c", "import tensorflow as tf; print(tf.test.gpu_device_name())"]
          resources:
            limits:
              nvidia.com/gpu: 1 

```

Now, if you deploy both YAML files. You will see two nodes provisioned in AWS console or you can see via use `` kubectl get nodes — show-labels``

## **Benefits of This Setup** 🎉

💸 **Cost Efficiency**: Dynamically scale GPU nodes up or down based on actual workload requirements, avoiding unnecessary costs when GPUs are idle.  

⚡ **Maximum Resource Utilization**: GPU time-slicing ensures all GPU resources are efficiently utilized, reducing wastage and boosting ROI.  

🔄 **Flexibility at Scale**: Combine spot and on-demand instances seamlessly to strike the perfect balance between performance and cost optimization.  

📊 **Workload-Aware Scaling**: Karpenter’s dynamic scaling adapts to real-time workload demands, ensuring your applications always have the resources they need.  

🔒 **Enhanced Isolation**: Technologies like NVIDIA MIG provide hardware-level isolation, ensuring secure and predictable GPU performance for sensitive workloads.  

---

## **Wrap-Up** 🚀

By integrating **GPU slicing** with **Karpenter Autoscaler**, we unlock a scalable, efficient, and cost-effective solution for GPU-intensive workloads. From training massive AI models to handling real-time inference, this setup ensures  maximization performance without affecting budget :) 
Thanks ! 

Sources and references : 
[GPU Nodes Time Slicing](https://medium.com/jina-ai/using-karpenter-to-manage-gpu-nodes-with-time-slicing-129098a72cb6).
[Eks Implementing GPU nodes with Nvidia drivers ](https://medium.com/@marcincuber/amazon-eks-implementing-and-using-gpu-nodes-with-nvidia-drivers-08d50fd637fe).





