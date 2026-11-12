# 01 - Conceptos de Contenedores

En este módulo se repasan los fundamentos de la contenerización para entender qué ejecuta Kubernetes por debajo antes de pasar a la orquestación.

## Material Teórico
* **[Descargar guía teórica del módulo (PDF)](./01-conceptos-contenedores.pdf)** — Apuntes completos sobre virtualización, arquitectura del kernel de Linux (*namespaces* y *cgroups*), funcionamiento de imágenes en capas y el modelo de Pods.

## Objetivos del módulo
* Diferenciar entre máquinas virtuales tradicionales y contenedores.
* Experimentar el aislamiento real de procesos y recursos en el kernel de Linux mediante *namespaces* y *cgroups*.
* Construir imágenes personalizadas utilizando un archivo `Dockerfile`.
* Crear un Pod multicontenedor con Podman para comprender el modelo que luego utiliza Kubernetes.

## Conceptos clave
* **Namespaces:** Característica del kernel de Linux que aísla lo que un proceso puede ver (árbol de procesos `PID`, interfaces de red `NET`, puntos de montaje `MNT`, nombre de host `UTS`).
* **Control Groups (cgroups):** Mecanismo del kernel que limita y mide los recursos físicos (CPU, memoria RAM, I/O) que un grupo de procesos puede consumir.
* **Imagen vs. Contenedor:** Una imagen es una plantilla inmutable en capas; el contenedor es una instancia en ejecución con una capa superior de lectura y escritura.
* **Docker:** Herramienta de gestión de contenedores, popular por poseer repositorios con gran variedad de imágenes de contenedor hardenizadas, y ser la pionera en masificar el uso de los contenedores.
* **Dockerfile:** Archivo de texto con instrucciones paso a paso para construir una imagen.
* **Container Runtime:** Motor encargado de ejecutar los contenedores (ej. `containerd`, `CRI-O`, Docker Engine).

## Comandos básicos de referencia
En estos comandos se puede reemplazar la invocación al binario "docker ..." por "podman ..." en caso de no tener instalado docker (caso de CentOS).
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
* **Podman:** Alternativa daemonless, contenedores rootless, CLI prácticamente idéntico a Docker, compatible con OCI, maneja PODS (como Kubernetes)
* **Containerd:** Motor de bajo nivel de Docker, proyecto de la CNCF, interacción más directa con el kernel para gestionar contenedores
* **Rancher Desktop:** Aplicación gráfica de código abierto que utiliza containerd o K3s, alternativa a Docker Desktop en varios OS
* **CRI-O:** Entorno de ejecución de contenedores ligero que se integra con Kubernetes, consumo mínimo de recursos, compatible OCI, ideal para entornos productivos con Kubernetes
* **LXC / LXD:** Virtualización a nivel de sistema operativo basada en contenedores Linux, ideal para gestionar entornos completos con servicios múltiples en lugar de contenedores de aplicación únicos

## Prácticas del módulo

### Ejercicio 1: Aislamiento de procesos en Linux con Namespaces y cgroups
*Requisitos: Sistema Linux con privilegios de administrador (`sudo`) y `systemd` (cgroups v2).*

1. **Aislamiento de procesos y hostname (`PID` y `UTS` namespaces):**
   Ejecutá una nueva sesión de `bash` aislada del sistema anfitrión:
   ```bash
   sudo unshare --pid --uts --mount-proc --fork bash
   ```
2. **Verificar el aislamiento dentro de la sesión:**
   ```bash
   hostname contenedor-manual
   hostname
   ps aux
   ```
   *Observación:* El proceso `bash` tiene el **PID 1** y no puede ver los procesos del sistema anfitrión. Escribí `exit` para salir.

3. **Límite de memoria con `cgroups` (v2 mediante `systemd-run`):**
   Lanzá un proceso limitado a **50 MB de RAM** que intente consumir 100 MB para observar cómo el kernel lo detiene (*OOM Killer*):
   ```bash
   sudo systemd-run --scope -p MemoryMax=50M -p MemorySwapMax=0 \
     python3 -c "a = bytearray(100 * 1024 * 1024)"
   ```
   *Observación:* El proceso terminará abruptamente con el mensaje `Killed` al superar el límite del *cgroup*.

---

### Ejercicio 2: Creación de una imagen con Dockerfile
*Ubicación del archivo: [ejemplos/ejercicio-02-dockerfile/](https://github.com/jomatoro/kubernetes-indra/tree/main/01-conceptos-contenedores/ejemplos/ejercicio-02-dockerfile)*

En este ejercicio se genera una imagen de contenedor basado en una imagen básica. Si no tenemos instalado docker, podemos modificar los comandos "docker ..." por "podman ..." (caso CentOS).  
El archivo Dockerfile de ejemplo se muestra a continuación:  
   ```bash
   FROM nginx:1.27-alpine
   # Copiar la página personalizada al directorio público de Nginx
   COPY index.html /usr/share/nginx/html/index.html
   EXPOSE 80
   CMD ["nginx", "-g", "daemon off;"]
   ```

1. **Ingresar a la carpeta del ejercicio:**
   ```bash
   cd ejemplos/ejercicio-02-dockerfile
   ```
2. **Construir la imagen:**
   ```bash
   docker build -t web-curso-k8s:1.0 .
   ```
   *(También funciona reemplazando `docker` por `podman`).*
3. **Ejecutar el contenedor y probarlo:**
   ```bash
   docker run -d --name mi-web -p 8080:80 web-curso-k8s:1.0
   curl http://localhost:8080
   ```
4. **Limpiar el entorno:**
   ```bash
   docker rm -f mi-web
   ```

---

### Ejercicio 3: Creación de un Pod de 2 contenedores con Podman

En este ejercicio un contenedor web (`nginx`) y un contenedor auxiliar (`busybox`) comparten el mismo *namespace* de red dentro de un Pod de Podman.

1. **Crear el Pod exponiendo el puerto 8081:**
   ```bash
   podman pod create --name pod-demo -p 8081:80
   ```
2. **Agregar el primer contenedor (servidor web Nginx) al Pod:**
   ```bash
   podman run -d --pod pod-demo --name cont-nginx nginx:1.27-alpine
   ```
3. **Agregar el segundo contenedor (cliente BusyBox) al mismo Pod:**
   ```bash
   podman run -d --pod pod-demo --name cont-helper busybox:1.36 sleep 3600
   ```
4. **Comprobar que comparten red (`localhost`):**
   Ejecutá `wget` desde el segundo contenedor apuntando a `localhost:80` y verificá que responde el servidor Nginx del primer contenedor:
   ```bash
   podman exec cont-helper wget -qO- http://localhost:80
   ```
5. **Opcional crear un yaml compatible con kubernetes a partir del pod creado:**
   ```bash
   podman generate kube pod-demo > pod-k8s.yaml
   ```
6. **Ver los contenedores agrupados y limpiar:**
   ```bash
   podman pod ps
   podman ps --pod
   podman pod rm -f pod-demo
   ```
   
