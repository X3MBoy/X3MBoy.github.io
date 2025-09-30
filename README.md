---
title: Ejercicio Práctico Docker y GAR
tags: gcp, google, cloud, docker
description: Ejercicio práctico de Docker y Google Artifact Registry
---

# Ejercicio Práctico: Docker, Podman y Google Artifact Registry (GAR)

## Objetivo
Crear una imagen Docker/Podman con una aplicación básica, subirla a Google Artifact Registry (GAR), analizarla y luego ejecutarla localmente desde el GAR.


## Requisitos Previos
- Tener instalado **Docker** o **Podman** (puedes elegir uno).
- Tener una cuenta en **Google Cloud Platform (GCP)** y el **SDK de Google Cloud (`gcloud`)** instalado y configurado.
- Acceso a **Google Artifact Registry** (habilitado en GCP).

## Pasos del Ejercicio

### 1. Crear una aplicación básica

Vamos a usar un simple servidor web en Python con Flask.

Crea un directorio para el proyecto:

```bash
mkdir app-gar && cd app-gar
```

Crea un archivo `app.py` con el siguiente contenido:

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello():
    return "¡Hola! Esta es mi imagen en Google Artifact Registry."

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8080)
```

Crea un archivo `requirements.txt` con las dependencias:

```
flask==2.0.1
```

### 2. Crear un Dockerfile
Crea un archivo `Dockerfile` con:

```dockerfile
# Usamos una imagen base de Python
FROM python:3.9-slim

# Copiamos la aplicación
WORKDIR /app
COPY . .

# Instalamos dependencias
RUN pip install --no-cache-dir -r requirements.txt

# Exponemos el puerto y ejecutamos la app
EXPOSE 8080
CMD ["python", "app.py"]
```

### 3. Construir la imagen
Usando **Docker**:

```bash
docker build -t my-flask-app .
```

Usando **Podman**:

```bash
podman build -t my-flask-app .
```

Verifica que la imagen se creó correctamente:

```bash
docker images  # o `podman images`
```

### 4. Configurar Google Artifact Registry (GAR)

Crea un repositorio en GAR (asegúrate de que la API esté habilitada):

```bash
gcloud artifacts repositories create my-repo \
  --repository-format=docker \
  --location=us-central1 \
  --description="Repositorio Docker para mi aplicación"
```

Configura Docker/Podman para autenticarse con GAR:

```bash
gcloud auth configure-docker us-central1-docker.pkg.dev
```

### 5. Subir la imagen a GAR
Primero, etiqueta la imagen con la URL de tu repositorio:

```bash
docker tag my-flask-app us-central1-docker.pkg.dev/[TU-PROYECTO-ID]/my-repo/my-flask-app:1.0
# o con Podman:
podman tag my-flask-app us-central1-docker.pkg.dev/[TU-PROYECTO-ID]/my-repo/my-flask-app:1.0
```

Sube la imagen:

```bash
docker push us-central1-docker.pkg.dev/[TU-PROYECTO-ID]/my-repo/my-flask-app:1.0
# o con Podman:
podman push us-central1-docker.pkg.dev/[TU-PROYECTO-ID]/my-repo/my-flask-app:1.0
```

Verifica en la consola de GCP (Artifact Registry) que la imagen esté disponible.

### 6. Analizar la imagen (opcional)
Puedes usar **Dive** o **Trivy** para analizar la imagen:

```bash
docker run --rm -it \
  -v /var/run/docker.sock:/var/run/docker.sock \
  wagoodman/dive:latest us-central1-docker.pkg.dev/[TU-PROYECTO-ID]/my-repo/my-flask-app:1.0
```

### 7. Descargar y ejecutar la imagen desde GAR
Descarga la imagen:

```bash
docker pull us-central1-docker.pkg.dev/[TU-PROYECTO-ID]/my-repo/my-flask-app:1.0
# o con Podman:
podman pull us-central1-docker.pkg.dev/[TU-PROYECTO-ID]/my-repo/my-flask-app:1.0
```

Ejecuta el contenedor:

```bash
docker run -d -p 8080:8080 us-central1-docker.pkg.dev/[TU-PROYECTO-ID]/my-repo/my-flask-app:1.0
# o con Podman:
podman run -d -p 8080:8080 us-central1-docker.pkg.dev/[TU-PROYECTO-ID]/my-repo/my-flask-app:1.0
```

Verifica en tu navegador o con `curl`:

```bash
curl http://localhost:8080
```
Deberías ver:

```
¡Hola! Esta es mi imagen en Google Artifact Registry.
```

## Conclusión

En este ejercicio:
✅ Creamos una aplicación básica en Python.
✅ Construimos una imagen con Docker/Podman.
✅ Subimos la imagen a Google Artifact Registry (GAR).
✅ Analizamos la imagen (opcional).
✅ Descargamos y ejecutamos la imagen desde GAR.