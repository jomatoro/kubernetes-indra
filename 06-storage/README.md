# 06 - Storage en Kubernetes

Los contenedores son efímeros por naturaleza. En este módulo verás cómo conservar datos de forma persistente más allá del ciclo de vida de un Pod.

## Objetivos del módulo
* Diferenciar entre volúmenes efímeros (`emptyDir`) y almacenamiento persistente.
* Aprovisionar almacenamiento de manera estática y dinámica.
* Entender las políticas de recuperación (*Reclaim Policies*).

## Conceptos clave
* **PersistentVolume (PV):** Recurso de almacenamiento físico o en la nube aprovisionado en el clúster.
* **PersistentVolumeClaim (PVC):** Solicitud de almacenamiento realizada por un usuario o aplicación con un tamaño y modo de acceso específico.
* **StorageClass:** Define perfiles de almacenamiento (ej. SSD, HDD) para crear volúmenes de forma dinámica automática.

## Comandos básicos de referencia
```bash
# Listar volúmenes persistentes, reclamos y clases de almacenamiento
kubectl get pv,pvc,sc

# Ver detalles y estado de enlace (Bound/Pending) de un PVC
kubectl describe pvc datos-db-pvc
```

## Práctica sugerida
Crear un `PersistentVolumeClaim`, montarlo en un Pod que escriba un archivo en esa ruta, eliminar el Pod y verificar en un nuevo Pod que el archivo sigue intacto.

# Ejercicios Prácticos de PV y PVC en Kubernetes (Sin Aprovisionamiento Dinámico)

Al no haber almacenamiento dinámico, **el administrador crea manualmente el PV antes que el PVC**, y debe apuntar explícitamente a un nodo específico (usando `local` con `nodeAffinity` o `hostPath`) para garantizar que el Pod siempre vuelva al nodo donde están sus datos físicos.

> **Nota:** En todos los ejercicios reemplaza `worker1` y `worker2` por los nombres reales de tus nodos obtenidos con `kubectl get nodes`.

---

## Ejercicio 1: El ciclo básico PV + PVC + Pod (`hostPath`)

**Objetivo:** Crear un volumen persistente manual, reclamarlo con un PVC, escribir un archivo desde un Pod, destruir el Pod y comprobar que el archivo sobrevive.

### 1. Preparar el directorio en `worker1`
Conéctate por SSH a `worker1` y crea la carpeta:
```bash
sudo mkdir -p /mnt/data-ej1
```

### 2. Crear el PV, el PVC y el Pod (`ej1-basico.yaml`)
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-ej1
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: "/mnt/data-ej1"
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-ej1
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-ej1
spec:
  nodeName: worker1 # Forzamos worker1 porque hostPath no tiene afinidad automática
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "echo 'Dato persistente 1' >> /datos/registro.txt && sleep 3600"]
    volumeMounts:
    - mountPath: "/datos"
      name: vol-ej1
  volumes:
  - name: vol-ej1
    persistentVolumeClaim:
      claimName: pvc-ej1
```

### 3. Verificación
```bash
kubectl apply -f ej1-basico.yaml
kubectl get pv,pvc # Ambos deben estar en estado 'Bound'
kubectl delete pod pod-ej1 # Borramos el Pod
# Vuelve a crear solo el Pod (o reaplica el YAML) y lee el archivo:
kubectl apply -f ej1-basico.yaml
kubectl exec pod-ej1 -- cat /datos/registro.txt
```

---

## Ejercicio 2: Volumen `local` con `nodeAffinity` (Afinidad automática al nodo)

**Objetivo:** Superar la limitación de `hostPath`. Con un volumen de tipo `local` y `nodeAffinity`, Kubernetes programa automáticamente el Pod en el nodo correcto sin que tengas que usar `nodeName`.

### 1. Preparar el directorio en `worker2`
En `worker2`, ejecuta:
```bash
sudo mkdir -p /mnt/data-ej2
```

### 2. Crear PV `local`, PVC y Deployment (`ej2-local.yaml`)
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-ej2
spec:
  capacity:
    storage: 2Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local-storage
  local:
    path: /mnt/data-ej2
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - worker2
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-ej2
spec:
  storageClassName: local-storage
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi # Pide 1Gi, se enlazará al PV de 2Gi porque es el único disponible
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-persistente
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-ej2
  template:
    metadata:
      labels:
        app: nginx-ej2
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
        volumeMounts:
        - name: web-data
          mountPath: /usr/share/nginx/html
      volumes:
      - name: web-data
        persistentVolumeClaim:
          claimName: pvc-ej2
```

### 3. Verificación
```bash
kubectl apply -f ej2-local.yaml
kubectl get pods -o wide # Verás que Kubernetes lo envió solo a worker2
```

---

## Ejercicio 3: Selección exacta mediante `selector` y `matchLabels`

**Objetivo:** Tienes varios PVs creados en el clúster (por ejemplo, uno rápido y uno lento) y necesitas que un PVC reclame **específicamente** uno de ellos y no cualquiera que cumpla con el tamaño.

### 1. Preparar carpetas en `worker1`
```bash
sudo mkdir -p /mnt/disco-lento /mnt/disco-rapido
```

### 2. Crear dos PVs y un PVC con selector (`ej3-selector.yaml`)
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-lento
  labels:
    tipo: hdd
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  storageClassName: manual
  hostPath:
    path: /mnt/disco-lento
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-rapido
  labels:
    tipo: nvme
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  storageClassName: manual
  hostPath:
    path: /mnt/disco-rapido
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-db-critica
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
  selector:
    matchLabels:
      tipo: nvme
```

### 3. Verificación
```bash
kubectl apply -f ej3-selector.yaml
kubectl get pv,pvc # Verifica que pvc-db-critica se enlazó a pv-rapido y dejó pv-lento Available
```

---

## Ejercicio 4: Recuperar un PV en estado `Released` (Política `Retain`)

**Objetivo:** Aprender qué ocurre cuando borras un PVC cuya política es `Retain`: el PV pasa a estado `Released` y **ningún nuevo PVC puede usarlo** hasta que limpies la referencia `claimRef` manualmente.

### 1. Simular el borrado accidental del PVC del Ejercicio 1
```bash
kubectl delete pod pod-ej1
kubectl delete pvc pvc-ej1
kubectl get pv pv-ej1 # Verás que el STATUS dice 'Released'
```

### 2. Intentar crear un nuevo PVC (`pvc-ej1-nuevo`) apuntando al mismo volumen
Si creas otro PVC igual, quedará atascado en `Pending` porque el `pv-ej1` todavía recuerda al viejo `pvc-ej1`.

### 3. Liberar el PV para reutilizarlo sin perder los datos de `/mnt/data-ej1`
Ejecuta este comando para borrar el campo `claimRef` del PV:
```bash
kubectl patch pv pv-ej1 --type json -p '[{"op": "remove", "path": "/spec/claimRef"}]'
```

### 4. Verificación
```bash
kubectl get pv pv-ej1 # Volverá a estado 'Available' y el nuevo PVC pasará a 'Bound'
```

---

## Ejercicio 5: Múltiples Pods compartiendo datos (`ReadWriteMany` simulado)

**Objetivo:** Desplegar 2 Pods (un escritor y un lector) compartiendo el mismo volumen en un mismo nodo. *(Nota: En entornos sin servidor NFS externo, `ReadWriteMany` funciona si ambos Pods caen en el mismo worker).*

### 1. Preparar carpeta en `worker1`
```bash
sudo mkdir -p /mnt/compartido
```

### 2. Crear PV `ReadWriteMany`, PVC y dos Pods en el mismo nodo (`ej5-compartido.yaml`)
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-compartido
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  storageClassName: manual
  hostPath:
    path: /mnt/compartido
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-compartido
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-escritor
spec:
  nodeName: worker1
  containers:
  - name: writer
    image: busybox
    command: ["sh", "-c", "while true; do date >> /shared/log.txt; sleep 5; done"]
    volumeMounts:
    - name: shared-vol
      mountPath: /shared
  volumes:
  - name: shared-vol
    persistentVolumeClaim:
      claimName: pvc-compartido
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-lector
spec:
  nodeName: worker1
  containers:
  - name: reader
    image: busybox
    command: ["sh", "-c", "tail -f /shared/log.txt"]
    volumeMounts:
    - name: shared-vol
      mountPath: /shared
  volumes:
  - name: shared-vol
    persistentVolumeClaim:
      claimName: pvc-compartido
```

### 3. Verificación
```bash
kubectl apply -f ej5-compartido.yaml
kubectl logs pod-lector # Verás en tiempo real la fecha que escribe pod-escritor cada 5 segundos
