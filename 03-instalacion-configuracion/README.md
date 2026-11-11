# 03 - Instalación, Configuración y Verificación de Clústeres

En este módulo aprenderás a levantar un clúster desde cero, configurar el acceso cliente y validar el estado de salud de los nodos.

## Objetivos del módulo
* Desplegar un entorno local de pruebas y entender el flujo de instalación con `kubeadm`.
* Administrar múltiples clústeres y contextos mediante el archivo `kubeconfig`.
* Verificar que los componentes del sistema estén operativos.

## Conceptos clave
* **kubeadm:** Herramienta oficial para inicializar un clúster de Kubernetes siguiendo buenas prácticas.
* **kubeconfig:** Archivo ubicado por defecto en `~/.kube/config` que guarda credenciales, certificados y direcciones de clústeres.
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

## Práctica sugerida
Inicializar un clúster local (con Kind, Minikube o `kubeadm`), instalar un plugin de red CNI y comprobar que todos los nodos figuren en estado `Ready`.
