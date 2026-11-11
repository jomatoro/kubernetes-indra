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
