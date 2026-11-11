# 07 - Seguridad y Acceso en Kubernetes

Este módulo cubre cómo proteger el acceso a la API del clúster, gestionar datos sensibles y endurecer la ejecución de los contenedores.

## Objetivos del módulo
* Controlar quién puede hacer qué en el clúster mediante RBAC.
* Separar configuraciones y credenciales del código de la aplicación.
* Aplicar restricciones de privilegios a nivel de Pod y contenedor.

## Conceptos clave
* **RBAC (Role-Based Access Control):** Sistema de permisos basado en `Role`/`ClusterRole` (qué acciones se permiten) y `RoleBinding`/`ClusterRoleBinding` (a quién se asignan).
* **ServiceAccount:** Identidad utilizada por procesos dentro de un Pod para interactuar con la API de Kubernetes.
* **ConfigMap y Secret:** Objetos para inyectar variables de entorno o archivos de configuración y datos sensibles.
* **SecurityContext:** Configuración de usuario UID, capacidades de Linux y sistema de archivos de solo lectura para un Pod.

## Comandos básicos de referencia
```bash
# Verificar si un usuario o ServiceAccount tiene permiso para una acción
kubectl auth can-i create deployments --as=system:serviceaccount:default:mi-sa

# Crear un Secret genérico desde la línea de comandos
kubectl create secret generic db-pass --from-literal=password=S3cr3t0!
```

## Práctica sugerida
Crear un `ServiceAccount` de solo lectura que únicamente pueda listar Pods en un *namespace* específico y validar sus permisos con `kubectl auth can-i`.
