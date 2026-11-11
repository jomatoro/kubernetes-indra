# 04 - Scheduling en Kubernetes

Este capítulo explica cómo el `kube-scheduler` decide en qué nodo ejecutar cada Pod y cómo controlar esa ubicación según las necesidades de la aplicación.

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
