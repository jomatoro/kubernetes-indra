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
