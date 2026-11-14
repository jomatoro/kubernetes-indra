# 02 - Arquitectura y Conceptos Básicos de Kubernetes

Este capítulo introduce los componentes internos que hacen funcionar un clúster de Kubernetes y los objetos fundamentales para desplegar aplicaciones.

## Material Teórico
* **[Descargar guía teórica del módulo (PDF)](./02-arquitectura-conceptos-kubernetes.pdf)** — Apuntes completos sobre conceptos basicos de la arquitectura de un cluster Kubernetes.

## Objetivos del módulo
* Identificar las piezas del *Control Plane* y de los *Worker Nodes*.
* Entender la estructura declarativa de un manifiesto YAML.
* Crear y gestionar Pods, ReplicaSets y Deployments.

## Conceptos clave
* **Control Plane:** Cerebro del clúster compuesto por `kube-apiserver`, `etcd`, `kube-scheduler` y `kube-controller-manager`.
* **Worker Node:** Nodo donde corren las cargas de trabajo mediante `kubelet`, `kube-proxy` y el *Container Runtime*.
* **Pod:** La unidad mínima desplegable en Kubernetes; agrupa uno o más contenedores que comparten red y almacenamiento.
* **Deployment:** Objeto que gestiona actualizaciones declarativas y el escalado de Pods a través de ReplicaSets.

## Comandos básicos de referencia
```bash
# Aplicar un manifiesto YAML
kubectl apply -f deployment.yaml

# Listar recursos en el namespace actual
kubectl get pods,deployments,rs

# Escalar la cantidad de réplicas de un Deployment
kubectl scale deployment nginx-deploy --replicas=3
```

## Aplicación práctica de un cluster Kubernetes
Usar un clúster de Kubernetes es recomendable principalmente cuando tienes una arquitectura de microservicios compleja, necesitas escalar automáticamente ante picos de tráfico y requieres alta disponibilidad sin tiempos de inactividad. No es una solución universal; para aplicaciones pequeñas o monolíticas, suele agregar una complejidad operativa innecesaria.  

**Casos de uso:**  
* **Arquitectura de microservicios:** Si tu sistema está dividido en decenas de servicios independientes que se comunican entre sí, Kubernetes simplifica el descubrimiento de servicios, el balanceo de carga y la gestión de configuraciones.
* **Escalabilidad dinámica y picos de tráfico:** Cuando la demanda de tu aplicación varía drásticamente (por ejemplo, en un e-commerce durante eventos de descuentos), el clúster puede crear o destruir contenedores de forma automática según el consumo de CPU o memoria.
* **Alta disponibilidad y autorrecuperación:** Si tu negocio no puede permitirse caídas, Kubernetes detecta cuando un contenedor o servidor falla y lo reinicia o mueve a otro nodo sano en segundos, sin intervención humana.
* **Despliegues continuos sin cortes (Zero-Downtime):** Facilita estrategias avanzadas como actualizaciones graduales (rolling updates) o pruebas canary, permitiéndote lanzar nuevas versiones del código varias veces al día sin interrumpir a los usuarios.
* **Portabilidad entre nubes (Multi-cloud o Híbrido):** Si deseas evitar depender de un único proveedor (AWS, Google Cloud, Azure) o quieres combinar servidores propios (on-premise) con la nube, Kubernetes ofrece una capa estándar para ejecutar tus aplicaciones en cualquier entorno.

**Casos no recomendados:**  
* **Aplicaciones monolíticas o proyectos pequeños:** Si tienes una aplicación sencilla con una base de datos y un tráfico predecible, mantener un clúster es matar moscas a cañonazos. En estos casos, es más barato y rápido usar un servidor tradicional (VPS) con Docker Compose o plataformas gestionadas como Heroku, Render o Railway.
* **Equipos reducidos sin experiencia en DevOps:** Kubernetes tiene una curva de aprendizaje alta y exige mantenimiento constante (seguridad, redes, actualizaciones). Si tu equipo es pequeño y está enfocado 100% en desarrollar funcionalidades de negocio, conviene usar servicios Serverless (como AWS Lambda o Cloud Run) para delegar la infraestructura.

## Prácticas del módulo
Se requiere la clonación del repositorio en su servidor local, y cambiarse al directorio kubernetes-indra:  
   ```bash
   git clone https://github.com/jomatoro/kubernetes-indra
   cd kubernetes-indra
   ```
## Desplegar una arquitectura simplificada para una aplicación web que requiere diferentes tipos de controladores según su función: 
* **Frontend:** Una app web sin estado que debe poder escalarse rápidamente. 
* **Base de Datos:** Un motor de almacenamiento que requiere identidad única y discos persistentes. 
* **Monitoreo:** Un agente que debe correr obligatoriamente en cada nodo del clúster. 
* **Migración:** Una tarea única que prepare la base de datos antes de que el sistema opere. 
* **Respaldo:** Una tarea automatizada que limpie temporales o haga backups cada minuto. 

### Ejercicio 1: Despliegue imperativo o declarativo de 3 réplicas de Nginx mediante un Deployment llamado web-frontend (carga stateless)
Tomar como ejemplo el archivo frontend-deployment.yaml y ejecutar su despliegue:
   ```bash
   kubectl apply -f frontend-deployment.yaml 
   kubectl get deployments 
   kubectl get pods -l app=frontend 
   ```
### Ejercicio 2: Despliegue de 2 réplicas de una base de datos PostgreSQL utilizando nombres de red estables, mediante un Statefulset (carga stateful)
Tomar como ejemplo el archivo db-statefulset.yaml y ejecutar su despliegue:
   ```
   kubectl apply -f db-statefulset.yaml 
   kubectl get statefulset 
   kubectl get pods -l app=postgres 
   ```
Verás db-0 y db-1 creándose secuencialmente 

### Ejercicio 3: Despliega un agente de recolección de logs (fluent-bit) que se ejecute automáticamente en todos los nodos del clúster, mediante un Daemonset.
Tomar como ejemplo el archivo log-daemonset.yaml y ejecutar su despliegue:
   ```bash
   kubectl apply -f log-daemonset.yaml 
   kubectl get daemonset 
   kubectl get pods -o wide 
   ```
Verás exactamente un pod por cada nodo de tu clúster 

