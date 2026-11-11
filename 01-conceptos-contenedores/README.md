# 01 - Conceptos de Contenedores

En este módulo se repasan los fundamentos de la contenerización para entender qué ejecuta Kubernetes por debajo antes de pasar a la orquestación.

## Objetivos del módulo
* Diferenciar entre máquinas virtuales tradicionales y contenedores.
* Comprender cómo funcionan los *namespaces* y *cgroups* en Linux.
* Construir imágenes y gestionar el ciclo de vida de un contenedor.

## Conceptos clave
* **Imagen vs. Contenedor:** Una imagen es una plantilla de solo lectura con el código y sus dependencias; el contenedor es una instancia en ejecución de esa imagen.
* **Dockerfile:** Archivo de texto con instrucciones paso a paso para construir una imagen.
* **Container Runtime:** Motor encargado de ejecutar los contenedores (ej. `containerd`, `CRI-O`, Docker Engine).

## Comandos básicos de referencia
```bash
# Construir una imagen a partir de un Dockerfile
docker build -t mi-app:1.0 .

# Ejecutar un contenedor en segundo plano exponiendo un puerto
docker run -d --name web -p 8080:80 nginx:alpine

# Listar contenedores activos y ver sus registros
docker ps
docker logs web
```
## Alternativas a Docker
Existen varias alternativas destacadas a Docker, siendo Podman una de las más populares. A continuación se detallan las mejores opciones según lo que se requiera:
* **PODMAN:** Alternativa daemonless, contenedores rootless, CLI prácticamente idéntico a Docker, compatible con OCI, maneja PODS (como Kubernetes)
* **Containerd:** Motor de bajo nivel de Docker, proyecto de la CNCF, interacción más directa con el kernel para gestionar contenedores
* **Rancher Desktop:** Aplicación gráfica de código abierto que utiliza containerd o K3s, alternativa a Docker Desktop en varios OS
* **CRI-O:** Entorno de ejecución de contenedores ligero que se integra con Kubernetes, consumo mínimo de recursos, compatible OCI, ideal para entornos productivos con Kubernetes
* **LXC / LXD:** Virtualización a nivel de sistema operativo basada en contenedores Linux, ideal para gestionar entornos completos con servicios múltiples en lugar de contenedores de aplicación únicos

## Práctica sugerida
Crear un `Dockerfile` sencillo para una aplicación web estática, construir la imagen localmente y verificar su funcionamiento en el navegador.

