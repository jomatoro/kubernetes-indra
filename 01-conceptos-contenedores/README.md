# 01 - Conceptos de Contenedores

En este módulo se repasan los fundamentos de la contenedorización para entender qué ejecuta Kubernetes por debajo antes de pasar a la orquestación.

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

## Práctica sugerida
Crear un `Dockerfile` sencillo para una aplicación web estática, construir la imagen localmente y verificar su funcionamiento en el navegador.

