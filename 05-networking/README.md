# 05 - Networking en Kubernetes

Aquí se aborda cómo se comunican los Pods entre sí, cómo exponer aplicaciones hacia afuera del clúster y cómo restringir el tráfico interno.

## Objetivos del módulo
* Comprender el modelo de red plano de Kubernetes.
* Exponer aplicaciones usando distintos tipos de `Services` e `Ingress`.
* Aislar tráfico entre aplicaciones mediante `NetworkPolicies`.

## Conceptos clave
* **Service (ClusterIP, NodePort, LoadBalancer):** Punto de acceso estable con IP fija y nombre DNS interno para un grupo dinámico de Pods.
* **CoreDNS:** Servidor DNS interno que resuelve nombres de servicios dentro del clúster.
* **Ingress:** Objeto que gestiona el acceso HTTP/HTTPS externo mediante rutas y dominios usando un *Ingress Controller*.
* **NetworkPolicy:** Firewall a nivel de Pod que controla el tráfico entrante (*Ingress*) y saliente (*Egress*).

## Comandos básicos de referencia
```bash
# Exponer un Deployment rápidamente como servicio ClusterIP
kubectl expose deployment web --port=80 --target-port=8080

# Listar servicios y sus endpoints asociados
kubectl get svc,endpoints

# Reenviar un puerto local hacia un servicio o Pod para pruebas
kubectl port-forward svc/web 8080:80
```

## Práctica sugerida
Desplegar un backend y un frontend, comunicarlos mediante DNS interno (`ClusterIP`) y crear una `NetworkPolicy` que bloquee el acceso desde otros *namespaces*.

---

## Laboratorio: ClusterIP, NodePort e Ingress NGINX

**Escenario:** Kubernetes *on-premises* sin salida cloud.  
**Objetivo:** Comprender la exposición de aplicaciones dentro y fuera del clúster.  
**Prerrequisito:** Tener el ingress nginx instalado y patcheado para soportar binding a los puertos del host con HostNetwork=true  
   ```bash
   # Instalar el ingress Nginx oficial desde github
   kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.15.1/deploy/static/provider/baremetal/deploy.yaml
   # Aplicar el patch para el binding a los nodos del host
   kubectl patch deployment ingress-nginx-controller -n ingress-nginx -p '{"spec":{"template":{"spec":{"hostNetwork":true,"dnsPolicy":"ClusterFirstWithHostNet"}}}}'
   ```

### 1. Arquitectura del laboratorio
Usaremos dos aplicaciones (`app1` y `app2`) desplegadas en un namespace dedicado (`networking-lab`) y las expondremos progresivamente mediante `ClusterIP`, `NodePort` e `Ingress`.

### 2. Validar Ingress Controller instalado
Primero verificamos que el **NGINX Ingress Controller** exista y esté corriendo en el clúster:

```bash
kubectl get pods -n ingress-nginx
```

Ejemplo esperado:
```text
NAME                                        READY
ingress-nginx-controller-xxxxx              1/1
```

Validar el servicio asociado:
```bash
kubectl get svc -n ingress-nginx
```

Ejemplo esperado:
```text
NAME                       TYPE
ingress-nginx-controller   NodePort
```

En un ambiente *on-premises* normalmente veremos los siguientes puertos mapeados:
* `80 -> 30080`
* `443 -> 30443`

El acceso externo será a través de:
```text
http://IP-NODO:30080
```

### 3. Crear namespace del laboratorio
Crear el archivo `namespace.yaml`:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: networking-lab
```

Aplicar y validar:
```bash
kubectl apply -f namespace.yaml
kubectl get namespaces
```

### 4. Crear primera aplicación nginx (`Deployment` `app1`)
Crear el archivo `app1-deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app1
  namespace: networking-lab
spec:
  replicas: 2
  selector:
    matchLabels:
      app: app1
  template:
    metadata:
      labels:
        app: app1
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

Aplicar y validar:
```bash
kubectl apply -f app1-deployment.yaml
kubectl get pods -n networking-lab
```

Resultado esperado:
```text
app1-xxxxx   Running
app1-yyyyy   Running
```

### 5. Exponer aplicación mediante `ClusterIP`
`ClusterIP` es el tipo de servicio por defecto y solo existe dentro del clúster.

Crear el archivo `app1-service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: app1-service
  namespace: networking-lab
spec:
  type: ClusterIP
  selector:
    app: app1
  ports:
  - port: 80
    targetPort: 80
```

Aplicar y validar:
```bash
kubectl apply -f app1-service.yaml
kubectl get svc -n networking-lab
```

Ejemplo esperado:
```text
NAME             TYPE
app1-service     ClusterIP
```

**Probar desde dentro del clúster:**  
Crear un Pod temporal interactivo:
```bash
kubectl run test --image=busybox -it --rm -- sh
```

Dentro de la consola del Pod, consultar el DNS interno del servicio:
```sh
wget -qO- app1-service.networking-lab.svc.cluster.local
```
*Debe responder el código HTML de bienvenida de nginx.*

### 6. Exponer aplicación con `NodePort`
Ahora habilitaremos la exposición para clientes externos mediante un puerto estático en los nodos.

Crear el archivo `app1-nodeport.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: app1-nodeport
  namespace: networking-lab
spec:
  type: NodePort
  selector:
    app: app1
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30081
```

Aplicar y validar:
```bash
kubectl apply -f app1-nodeport.yaml
kubectl get svc -n networking-lab
```

Salida esperada:
```text
app1-nodeport   NodePort   80:30081
```

Desde fuera del clúster, acceder en el navegador o con `curl` a:
```text
http://IP_WORKER:30081
```
*Debe aparecer la página de nginx.*

### 7. Crear segunda aplicación (`Deployment` `app2`)
Crear el archivo `app2.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app2
  namespace: networking-lab
spec:
  replicas: 2
  selector:
    matchLabels:
      app: app2
  template:
    metadata:
      labels:
        app: app2
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

Aplicar el Deployment:
```bash
kubectl apply -f app2.yaml
```

Crear el servicio `ClusterIP` en el archivo `app2-service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: app2-service
  namespace: networking-lab
spec:
  selector:
    app: app2
  ports:
  - port: 80
    targetPort: 80
```

Aplicar el Service:
```bash
kubectl apply -f app2-service.yaml
```

### 8. Crear `Ingress`
Ahora centralizamos el acceso HTTP por nombre de dominio para evitar publicar múltiples puertos `NodePort`.

Crear el archivo `apps-ingress.yaml`:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: apps-ingress
  namespace: networking-lab
spec:
  ingressClassName: nginx
  rules:
  - host: app1.lab.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app1-service
            port:
              number: 80
  - host: app2.lab.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app2-service
            port:
              number: 80
```

Aplicar el Ingress:
```bash
kubectl apply -f apps-ingress.yaml
```

### 9. Configurar DNS local
Como no contamos con un servidor DNS corporativo en el laboratorio, editamos el archivo local de resolución de nombres:

```bash
sudo vi /etc/hosts
```

Agregar las siguientes entradas (donde `192.168.56.101` es la IP del nodo que expone el Ingress Controller):
```text
192.168.56.101 app1.lab.local
192.168.56.101 app2.lab.local
```

### 10. Probar acceso
Abrir en el navegador (o consultar con `curl`, agregando `:30080` si el Ingress Controller escucha en ese NodePort):
* `http://app1.lab.local` *(Debe responder: **Welcome to nginx**)*
* `http://app2.lab.local` *(Debe responder: **Welcome to nginx**)*

### Resultado esperado del laboratorio
Al finalizar se comprende el rol de cada componente en el flujo de red:

| Objeto | Función |
| :--- | :--- |
| **Pod** | Ejecuta aplicación |
| **Deployment** | Mantiene réplicas |
| **ClusterIP** | Comunicación interna |
| **NodePort** | Publicación básica externa |
| **Ingress Controller** | Proxy reverso dentro del clúster |
| **Ingress Resource** | Reglas HTTP/HTTPS |

