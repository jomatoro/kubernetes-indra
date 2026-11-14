# 03 - Instalación, Configuración y Verificación de Clústeres

En este módulo aprenderás a levantar un clúster desde cero, configurar el acceso cliente y validar el estado de salud de los nodos.

## Material Teórico
* **[Descargar guía teórica del módulo (PDF)](./03-instalacion-configuracion.pdf)** — Apuntes completos sobre instalación, configuración y verificación de un clúster Kubernetes basado en la herramienta kubeadm.

## Objetivos del módulo
* Desplegar un entorno local de pruebas y entender el flujo de instalación con `kubeadm`.
* Administrar múltiples clústeres y contextos mediante el archivo `kubeconfig`.
* Verificar que los componentes del sistema estén operativos.

## Conceptos clave
* **kubeadm:** Herramienta oficial para inicializar un clúster de Kubernetes siguiendo buenas prácticas.
* **kubeconfig:** Archivo ubicado por defecto en `~/.kube/config` que guarda credenciales, certificados y direcciones de clústeres.
* **CRI (Container Runtime Interface):** Conjunto de estándares y protocolos en Kubernetes que permite al componente kubelet comunicarse con los motores que ejecutan los contenedores.
* **CNI (Container Network Interface):** Plugin de red necesario para que los nodos pasen al estado `Ready` (ej. Calico, Flannel).

## Comandos básicos de referencia
```bash
# Verificar el estado de los nodos y componentes del sistema
kubectl get nodes -o wide
kubectl get pods -n kube-system

# Ver y cambiar el contexto activo de kubectl
kubectl config get-contexts
kubectl config use-context nombre-del-contexto

# Obtener información general del clúster
kubectl cluster-info
```

## Prácticas del módulo
Se requiere la clonación del repositorio en su servidor local, y cambiarse al directorio kubernetes-indra:  
   ```bash
   git clone https://github.com/jomatoro/kubernetes-indra
   cd kubernetes-indra
   ```


## Práctica sugerida
Inicializar un clúster local (con Kind, Minikube o `kubeadm`), instalar un plugin de red CNI y comprobar que todos los nodos figuren en estado `Ready`.
