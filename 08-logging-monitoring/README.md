# 08 - Logging y Monitoring en Kubernetes

En este capítulo aprenderás a obtener visibilidad sobre el comportamiento del clúster y el estado de salud de tus aplicaciones en ejecución.

## Objetivos del módulo
* Consultar registros de contenedores actuales y reiniciados.
* Monitorear el consumo de CPU y memoria en tiempo real.
* Configurar sondas de salud (*Probes*) para mantener alta disponibilidad.

## Conceptos clave
* **Metrics Server:** Componente ligero que recolecta métricas de recursos de nodos y Pods para habilitar `kubectl top` y el autoescalado (HPA).
* **Liveness, Readiness y Startup Probes:** Comprobaciones periódicas que le indican a Kubernetes cuándo reiniciar un contenedor o cuándo está listo para recibir tráfico.
* **Arquitectura de Logging:** Recolección centralizada de la salida estándar (`stdout` y `stderr`) mediante agentes a nivel de nodo.

## Comandos básicos de referencia
```bash
# Ver consumo actual de CPU y memoria de nodos y Pods
kubectl top nodes
kubectl top pods -A

# Consultar logs en vivo o de una instancia anterior que falló
kubectl logs -f nombre-pod
kubectl logs nombre-pod --previous
```

## Práctica sugerida
Configurar un `Deployment` con `readinessProbe` y `livenessProbe` HTTP, simular una falla en la ruta de salud y observar los eventos de reinicio del contenedor.
