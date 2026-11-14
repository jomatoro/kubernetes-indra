# 03 - Instalación, Configuración y Verificación de Clústeres

En este módulo aprenderás a levantar un clúster desde cero, configurar el acceso cliente y validar el estado de salud de los nodos.

## Material Teórico
* **[Descargar guía teórica del módulo (PDF)](./03-instalacion-configuracion-kubernetes.pdf)** — Apuntes completos sobre instalación, configuración y verificación de un clúster Kubernetes basado en la herramienta kubeadm.

## Objetivos del módulo
* Desplegar un entorno local de pruebas y entender el flujo de instalación con `kubeadm`.
* Administrar múltiples clústeres y contextos mediante el archivo `kubeconfig`.
* Verificar que los componentes del sistema estén operativos.

## Conceptos clave
* **kubeadm:** Herramienta oficial para inicializar un clúster de Kubernetes siguiendo buenas prácticas.
* **kubeconfig:** Archivo ubicado por defecto en `~/.kube/config` que guarda credenciales, certificados y direcciones de clústeres.
* **CRI (Container Runtime Interface):** Conjunto de estándares y protocolos en Kubernetes que permite al componente kubelet comunicarse con los motores que ejecutan los contenedores.
* **CNI (Container Network Interface):** Plugin de red necesario para que los nodos pasen al estado `Ready` (ej. Calico, Flannel).

## Comandos básicos de referencia
```bash
# Verificar el estado de los nodos y componentes del sistema
kubectl get nodes -o wide
kubectl get pods -n kube-system

# Ver y cambiar el contexto activo de kubectl
kubectl config get-contexts
kubectl config use-context nombre-del-contexto

# Obtener información general del clúster
kubectl cluster-info
```

## Prácticas del módulo
Se requiere la clonación del repositorio en su servidor local, y cambiarse al directorio kubernetes-indra:  
   ```bash
   git clone https://github.com/jomatoro/kubernetes-indra
   cd kubernetes-indra
   ```
## Experimentar la diferencia entre los tres efectos principales de los Taints.
* **NoSchedule:** Si el Pod no tolera el taint, no se planifica en el nodo (los que ya están corriendo no se ven afectados).
* **PreferNoSchedule:** El sistema intenta evitar el nodo, pero si no hay espacio en otro lado, lo termina metiendo ahí.
* **NoExecute:** Si el Pod no tolera el taint, es expulsado (evicted) inmediatamente del nodo, incluso si ya estaba corriendo.  

Necesitamos saber el nombre de un nodo para aplicarle las reglas, para ello ejecutar ```kubectl get nodes```. Para este ejemplo, asumiremos que el nodo se llama worker1.

### Ejercicio 1: El efecto NoSchedule (Bloqueo de nuevas altas)
* Paso 1: Vamos a aplicarle un taint de tipo NoSchedule a nuestro nodo.
   ```bash
   kubectl taint nodes worker1 clave-prueba=bloqueo:NoSchedule
   ```
* Paso 2: Intenta crear un Pod común y corriente sin ninguna tolerancia. Aplicar el archivo pod-normal.yaml:
   ```bash
   kubectl apply -f pod-normal.yaml
   # Revisa el estado del Pod:
   kubectl get pods
   ```
* Resultado esperado: Verás que el Pod se queda en estado Pending. Si ejecutas kubectl describe pod pod-normal, en la sección de eventos abajo de todo verás un mensaje indicando que el nodo tenía un Taint que el Pod no toleraba (1 node(s) had untolerated taint).

### Ejercicio 2: El efecto PreferNoSchedule (El último recurso)
Paso 1: Primero, limpiemos el escenario anterior removiendo el taint (se quita agregando un signo menos - al final).
   ```bash
   kubectl delete pod pod-normal
   kubectl taint nodes worker1 clave-prueba:NoSchedule-
   ```
Paso 2: Ahora aplicamos el efecto "suave" (PreferNoSchedule):
   ```bash
   kubectl taint nodes worker1 clave-prueba=preferencia:PreferNoSchedule
   ```
Paso 3: Vuelve a aplicar el mismo archivo pod-normal.yaml de antes:
   ```bash
   kubectl apply -f pod-normal.yaml
   # monitorea el estado:
   kubectl get pods
   ```
* Resultado esperado: El Pod pasará a Running. Al ser un entorno de un solo nodo (laboratorio local), el Scheduler no tiene otra opción disponible. Como PreferNoSchedule es una preferencia y no una obligación, Kubernetes prefiere romper la regla antes que dejar tu Pod sin ejecutar.

### Ejercicio 3: El efecto NoExecute (La expulsión inmediata)
Este es el efecto más agresivo. Vamos a ver cómo desaloja aplicaciones vivas.
Paso 1: Limpiemos el clúster nuevamente.
kubectl delete pod pod-normal
kubectl taint nodes minikube clave-prueba:PreferNoSchedule-
Paso 2: Vamos a crear un Deployment con 3 réplicas idénticas para ver el comportamiento en masa. Guarda esto como deploy-test.yaml:
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-test
spec:
  replicas: 3
  selector:
    matchLabels:
      app: mi-web
  template:
    metadata:
      labels:
        app: mi-web
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
kubectl apply -f deploy-test.yaml
Verifica que las 3 réplicas estén corriendo exitosamente (kubectl get pods).
Paso 3: Con los Pods ya ejecutándose, vamos a clavar el Taint NoExecute en el nodo:
kubectl taint nodes minikube clave-prueba=peligro:NoExecute
Paso 4: Mira inmediatamente qué pasa con tus Pods:
kubectl get pods
•	Resultado esperado: Los Pods existentes entran en estado Terminating de inmediato y desaparecen. El Deployment intentará crear nuevos Pods para cumplir con la cuota de 3, pero los nuevos se quedarán infinitamente en Pending porque el nodo ahora los expulsa.

## Limpieza Final del Laboratorio
Para dejar tu clúster limpio y como estaba al principio, remueve el último taint:
   ```bash
   kubectl delete deploy app-test
   kubectl taint nodes worker1 clave-prueba:NoExecute-
   ```
