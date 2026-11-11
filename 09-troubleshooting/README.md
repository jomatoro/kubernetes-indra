# 09 - Troubleshooting en Kubernetes

El módulo final reúne una metodología sistemática para diagnosticar y resolver fallos frecuentes en aplicaciones, redes y nodos del clúster.

## Objetivos del módulo
* Interpretar estados de error habituales en Pods (`CrashLoopBackOff`, `ImagePullBackOff`, `Pending`).
* Diagnosticar fallos de conectividad entre Servicios y Pods.
* Revisar el estado de los demonios del sistema en nodos defectuosos.

## Conceptos clave
* **Flujo de diagnóstico de aplicaciones:** Revisar estado del Pod -> leer eventos (`describe`) -> inspeccionar registros (`logs`) -> verificar configuración y red.
* **Errores comunes:**
  * `Pending`: Falta de recursos en nodos, *taints* sin tolerar o PVC sin enlazar.
  * `ImagePullBackOff`: Nombre de imagen incorrecto, etiqueta inexistente o falta de credenciales de registro privado.
  * `CrashLoopBackOff`: La aplicación interna falla al iniciar o falla una *Liveness Probe*.
* **Diagnóstico de nodos:** Inspección del servicio `kubelet` y certificados del sistema operativo.

## Comandos básicos de referencia
```bash
# Ver eventos recientes ordenados por marca de tiempo
kubectl get events --sort-by='.lastTimestamp'

# Inspeccionar a fondo un Pod con fallos
kubectl describe pod pod-con-error

# Ejecutar un Pod temporal de red para depurar DNS y conectividad
kubectl run net-debug --rm -it --image=busybox:1.36 -- sh
```

## Práctica sugerida
Desplegar manifiestos con errores intencionales (puerto de servicio desalineado con `targetPort`, imagen inexistente y límites de memoria insuficientes) y resolver cada escenario paso a paso.
