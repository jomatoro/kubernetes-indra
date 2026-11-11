# 02 - Arquitectura y Conceptos Básicos de Kubernetes

Este capítulo introduce los componentes internos que hacen funcionar un clúster de Kubernetes y los objetos fundamentales para desplegar aplicaciones.

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

## Práctica sugerida
Escribir un manifiesto YAML para un `Deployment` de 3 réplicas de Nginx, desplegarlo y eliminar manualmente un Pod para observar cómo el controlador lo recrea automáticamente.
