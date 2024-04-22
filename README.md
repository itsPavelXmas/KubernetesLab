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

# Introducere in Kubernetes

Kubernetes (prescurtat K8s) este cel mai popular orchestrator, folosit la scara largă și oferit ca și CaaS (Container as a Service) de toți vendorii de infrastructură (Amazon, Google, Microsoft). Este considerat standard în materie de deployment al serviciilor orchestrate.
Kubernetes simplifică sarcinile complexe de gestionare a aplicațiilor containerizate, oferind mecanisme robuste pentru implementare, întreținere și scalare.


## **Concepte cheie Kubernetes**

