# 04 - Scheduling en Kubernetes

Este capítulo explica cómo el `kube-scheduler` decide en qué nodo ejecutar cada Pod y cómo controlar esa ubicación según las necesidades de la aplicación.

## Material Teórico
* **[Descargar guía teórica del módulo (PDF)](./04-scheduling.pdf)** — Conceptos teóricos sobre scheduling y labels en un clúster Kubernetes.

## Objetivos del módulo
* Organizar recursos utilizando etiquetas (*Labels*) y selectores.
* Restringir o atraer Pods hacia nodos específicos.
* Configurar solicitudes (*Requests*) y límites (*Limits*) de CPU y memoria.

## Conceptos clave
* **Labels y Selectors:** Pares clave-valor que permiten agrupar y filtrar objetos en Kubernetes.
* **Taints y Tolerations:** Permiten que un nodo rechace Pods a menos que estos tengan una tolerancia explícita.
* **Node Affinity:** Reglas avanzadas para indicar preferencias o requisitos obligatorios sobre dónde ubicar un Pod.
* **Resource Requests y Limits:** Garantizan recursos mínimos al Pod y evitan que consuma toda la capacidad del nodo.

## Comandos básicos de referencia
```bash
# Etiquetar un nodo
kubectl label nodes worker-01 disco=ssd

# Aplicar un Taint a un nodo para reservarlo
kubectl taint nodes worker-01 dedicado=gpu:NoSchedule

# Ver los recursos asignados y consumidos por nodo
kubectl describe node worker-01
```

## Práctica sugerida
Etiquetar un nodo como `entorno=produccion` y desplegar un Pod con `nodeSelector` y límites de memoria definidos para confirmar en qué nodo se programa.

---

## Laboratorios Prácticos

Antes de empezar, verificá los nombres de tus nodos para reemplazarlos en los comandos:
```bash
kubectl get nodes
```

### 1. Resource Requests y Limits (Throttling y OOMKilled)
Este ejemplo demuestra cómo el scheduler filtra nodos según los requests y qué ocurre cuando un contenedor supera el límite de memoria (`OOMKilled`).

**Paso 1:** Creá un pod que solicite más memoria de la que existe o que exceda su límite intencionalmente (`pod-oom.yaml`):
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-limits
spec:
  containers:
  - name: stress-mem
    image: polinux/stress
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "200M", "--vm-hang", "1"]
```

**Paso 2:** Aplicalo y observá cómo Kubernetes termina el contenedor por exceder los 128Mi de límite:
```bash
kubectl apply -f pod-oom.yaml
kubectl get pod test-limits -w
```

**Limpieza:**
```bash
kubectl delete pod test-limits
```

### 2. Atracción básica con nodeSelector
Asignaremos una etiqueta de hardware simulada al `worker-1` para forzar que un pod se ejecute únicamente ahí.

**Paso 1:** Etiquetá tu primer nodo worker:
```bash
kubectl label nodes worker-1 disktype=ssd
```

**Paso 2:** Desplegá un pod con `nodeSelector` (`pod-ssd.yaml`):
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-ssd
spec:
  nodeSelector:
    disktype: ssd
  containers:
  - name: nginx
    image: nginx:alpine
```

**Paso 3:** Verificá que cayó exclusivamente en `worker-1`:
```bash
kubectl apply -f pod-ssd.yaml
kubectl get pod pod-ssd -o wide
```

**Limpieza:**
```bash
kubectl delete pod pod-ssd && kubectl label nodes worker-1 disktype-
```

### 3. Repulsión y acceso exclusivo con Taints y Tolerations
Configuraremos el `worker-2` como un nodo dedicado para cargas especiales, repeliendo cualquier pod estándar.

**Paso 1:** Aplicá un taint `NoSchedule` al `worker-2`:
```bash
kubectl taint nodes worker-2 dedicated=especial:NoSchedule
```

**Paso 2:** Desplegá dos pods: uno normal (será repelido hacia `worker-1`) y otro con la toleration (`taints-demo.yaml`):
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-comun
spec:
  containers:
  - name: nginx
    image: nginx:alpine
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-tolerante
spec:
  tolerations:
  - key: "dedicated"
    operator: "Equal"
    value: "especial"
    effect: "NoSchedule"
  nodeSelector:
    kubernetes.io/hostname: worker-2
  containers:
  - name: nginx
    image: nginx:alpine
```

**Paso 3:** Comprobá la ubicación de ambos pods:
```bash
kubectl apply -f taints-demo.yaml
kubectl get pods -o wide
```

**Limpieza:**
```bash
kubectl delete -f taints-demo.yaml && kubectl taint nodes worker-2 dedicated=especial:NoSchedule-
```

### 4. Alta Disponibilidad con podAntiAffinity
Evitaremos que dos réplicas de una misma aplicación web se ejecuten en el mismo nodo worker.

**Paso 1:** Creá un Deployment de 3 réplicas con `podAntiAffinity` estricto por hostname (`ha-web.yaml`):
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-ha
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-store
  template:
    metadata:
      labels:
        app: web-store
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - web-store
            topologyKey: kubernetes.io/hostname
      containers:
      - name: nginx
        image: nginx:alpine
```

**Paso 2:** Observá qué pasa al tener solo 2 nodos workers disponibles: 2 pods pasarán a `Running` (uno en cada worker) y el tercer pod quedará en `Pending` por falta de un tercer nodo disponible.
```bash
kubectl apply -f ha-web.yaml
kubectl get pods -l app=web-store -o wide
```

**Limpieza:**
```bash
kubectl delete -f ha-web.yaml
```

### 5. Distribución uniforme con TopologySpreadConstraints (maxSkew)
Simularemos dos zonas de disponibilidad en tus workers y distribuiremos 4 pods equitativamente.

**Paso 1:** Asigná etiquetas de zona a cada worker:
```bash
kubectl label nodes worker-1 topology.kubernetes.io/zone=zona-a
kubectl label nodes worker-2 topology.kubernetes.io/zone=zona-b
```

**Paso 2:** Desplegá 4 réplicas con `maxSkew: 1` (`spread-demo.yaml`):
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-spread
spec:
  replicas: 4
  selector:
    matchLabels:
      app: spread-test
  template:
    metadata:
      labels:
        app: spread-test
    spec:
      topologySpreadConstraints:
      - maxSkew: 1
        topologyKey: topology.kubernetes.io/zone
        whenUnsatisfiable: DoNotSchedule
        labelSelector:
          matchLabels:
            app: spread-test
      containers:
      - name: nginx
        image: nginx:alpine
```

**Paso 3:** Verificá que haya exactamente 2 pods en `worker-1` y 2 pods en `worker-2`:
```bash
kubectl apply -f spread-demo.yaml
kubectl get pods -l app=spread-test -o wide
```

**Limpieza:**
```bash
kubectl delete -f spread-demo.yaml && kubectl label nodes worker-1 worker-2 topology.kubernetes.io/zone-
```

### 6. Protección ante mantenimiento con PodDisruptionBudget (PDB)
Protegeremos una aplicación crítica para que Kubernetes impida drenar un nodo si eso deja al servicio por debajo del mínimo disponible.

**Paso 1:** Creá un Deployment de 2 réplicas distribuido en ambos workers y un PDB con `minAvailable: 2` (`pdb-demo.yaml`):
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-critica
spec:
  replicas: 2
  selector:
    matchLabels:
      app: api-critica
  template:
    metadata:
      labels:
        app: api-critica
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchLabels:
                app: api-critica
            topologyKey: kubernetes.io/hostname
      containers:
      - name: nginx
        image: nginx:alpine
---
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: api-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: api-critica
```

**Paso 2:** Aplicá el manifiesto e intentá drenar el `worker-1` para simular un mantenimiento:
```bash
kubectl apply -f pdb-demo.yaml
kubectl drain worker-1 --ignore-daemonsets --delete-emptydir-data
```

**Resultado esperado:** El comando `drain` quedará bloqueado mostrando un error de violación de presupuesto (`Cannot evict pod as it would violate the pod's disruption budget`), protegiendo la disponibilidad del servicio.

**Limpieza (muy importante reactivar el nodo):**
```bash
kubectl uncordon worker-1
kubectl delete -f pdb-demo.yaml
```

