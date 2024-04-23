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

Vom utiliza o aplicatie simpla cu rolul de containerizare

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

Dockerfile
```
FROM node:14-alpine3.16 

WORKDIR /app 

COPY . .

RUN npm install

CMD [ "npm", "start"]

```



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




