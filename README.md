# Curso de Kubernetes

Bienvenido al repositorio oficial del curso de **Kubernetes**. Aquí encontrarás el material teórico, manifiestos YAML, ejemplos prácticos y laboratorios organizados módulo por módulo para aprender a desplegar, administrar y solucionar problemas en clústeres de contenedores en producción.

## Requisitos previos
Para aprovechar al máximo las prácticas de este repositorio, se recomienda contar con:
* Conocimientos básicos de línea de comandos en Linux.
* Nociones fundamentales de redes (IP, puertos, DNS).
* Tener instalado **Docker**, **kubectl** y una herramienta de clúster local como **Minikube** o **Kind** (se ve en detalle en el Módulo 03).

## Contenido del Curso

| Módulo | Tema | Descripción breve |
| :--- | :--- | :--- |
| **01** | [Conceptos de Contenedores](./01-conceptos-contenedores) | Fundamentos de virtualización, imágenes, contenedores y runtime. |
| **02** | [Arquitectura y Conceptos Básicos](./02-arquitectura-conceptos-basicos) | Componentes del Control Plane, Worker Nodes, Pods, ReplicaSets y Deployments. |
| **03** | [Instalación, Configuración y Verificación](./03-instalacion-configuracion) | Despliegue de clústeres, uso de `kubeadm`, configuración de `kubectl` y validación. |
| **04** | [Scheduling en Kubernetes](./04-scheduling) | Asignación de Pods a nodos, Labels, Selectors, Taints, Tolerations y Affinity. |
| **05** | [Networking en Kubernetes](./05-networking) | Modelo de red, Services (ClusterIP, NodePort, LoadBalancer), Ingress y Network Policies. |
| **06** | [Storage en Kubernetes](./06-storage) | Volúmenes efímeros, PersistentVolumes (PV), PersistentVolumeClaims (PVC) y StorageClasses. |
| **07** | [Seguridad y Acceso](./07-seguridad-acceso) | Autenticación, autorización con RBAC, ServiceAccounts y Security Contexts. |
| **08** | [Logging y Monitoring](./08-logging-monitoring) | Recolección de logs de contenedores, métricas del clúster y observabilidad. |
| **09** | [Troubleshooting en Kubernetes](./09-troubleshooting) | Diagnóstico y resolución de fallos en aplicaciones, nodos y componentes de red. |

## Estructura del Repositorio

Cada carpeta de capítulo sigue una estructura uniforme para facilitar el estudio:

```text
├── 01-conceptos-contenedores/
│   ├── README.md          # Apuntes y guía del módulo
│   └── ejemplos/          # Archivos de práctica y ejercicios
├── 02-arquitectura-conceptos-basicos/
...
└── 09-troubleshooting/
```

## Cómo usar este repositorio

1. **Cloná el repositorio en tu máquina local:**
   ```bash
   git clone https://github.com/jomatoro/kubernetes-indra
   ```
2. **Ingresá al directorio del curso:**
   ```bash
   cd curso-kubernetes
   ```
3. **Navegá al capítulo que quieras practicar:**
   ```bash
   cd 02-arquitectura-conceptos-basicos
   ```

## Licencia
Este material se distribuye bajo la Licencia MIT. Consultá el archivo `LICENSE` para más información.
