## 📌 **Índice**

- [[#🧠 1 Qué es Docker y para qué se usa]]
- [[#📄 **2. Dockerfile — estructura y secciones clave**]]
- [[#🏗️ 3 Construcción de imágenes con docker build]]
- [[#📥 4 Gestión de imágenes Docker]]
- [[#🚀 5 Despliegue de contenedores con docker run]]
- [[#📋 6 Gestión y control de contenedores]]
- [[#🧪 7 Ejecución de comandos dentro de contenedores]]
- [[#🧹 8 Eliminación de contenedores e imágenes]]
- [[#🌐 9 Port Forwarding en Docker]]
- [[#📂 10 Monturas volúmenes y persistencia]]
- [[#🧩 11 Introducción a Docker Compose]]


---

## 🧠 **1. Qué es Docker y para qué se usa**

Docker es una plataforma de **contenedorización** que permite empaquetar aplicaciones junto con:

- Sus dependencias
    
- Su configuración
    
- Su entorno de ejecución
    

Todo dentro de **contenedores aislados**, reproducibles y portables.

En hacking, desarrollo y labs se usa para:

- Levantar entornos vulnerables
    
- Replicar infraestructuras reales
    
- Probar exploits sin ensuciar el sistema
    
- Orquestar múltiples servicios (web + db + api)
    

---

## 📄 **2. Dockerfile — estructura y secciones clave**

Un **Dockerfile** define cómo se construye una imagen Docker.  
Está compuesto por **instrucciones en mayúsculas**.

### 🔹 Instrucciones más comunes

#### 🧱 FROM

Define la imagen base:

```dockerfile
FROM ubuntu:20.04
```

---

#### ⚙️ RUN

Ejecuta comandos dentro de la imagen durante la construcción:

```dockerfile
RUN apt update && apt install -y apache2
```

---

#### 📂 COPY

Copia archivos desde el host al contenedor:

```dockerfile
COPY index.html /var/www/html/
```

---

#### ▶️ CMD

Define el comando que se ejecuta al iniciar el contenedor:

```dockerfile
CMD ["apachectl", "-D", "FOREGROUND"]
```

⚠️ `CMD` se ejecuta **al arrancar el contenedor**, no al construir la imagen.

---

## 🏗️ **3. Construcción de imágenes con docker build**

Para construir una imagen:

```bash
docker build .
```

Etiqueta la imagen con nombre y versión:

```bash
docker build -t mi_imagen:v1 .
```

Si el Dockerfile está en otro directorio:

```bash
docker build -t mi_imagen:v1 /home/usuario/proyecto/
```

📌 Docker usa **caché por capas**, lo que acelera builds posteriores.

---

## 📥 **4. Gestión de imágenes Docker**

### Descargar una imagen desde Docker Hub

```bash
docker pull ubuntu:latest
```

---

### Listar imágenes locales

```bash
docker images
```

---

## 🚀 **5. Despliegue de contenedores con docker run**

El comando clave para crear y ejecutar contenedores.

### Sintaxis básica

```bash
docker run [opciones] imagen
```

Ejemplo típico:

```bash
docker run -dit mi_imagen
```

### Opciones importantes

- `-d` → modo background (detach)
    
- `-i` → modo interactivo
    
- `-t` → pseudo-terminal
    
- `--name` → nombre del contenedor
    

Ejemplo completo:

```bash
docker run -dit --name mycontainer mi_imagen
```

---

## 📋 **6. Gestión y control de contenedores**

### Ver contenedores activos

```bash
docker ps
```

### Ver todos (activos + detenidos)

```bash
docker ps -a
```

---

## 🧪 **7. Ejecutar comandos dentro de un contenedor**

Para acceder a un contenedor en ejecución:

```bash
docker exec -it mycontainer bash
```

O por ID:

```bash
docker exec -it 123456789 bash
```

Esto es equivalente a **“meterte dentro” del contenedor**.

---

## 🧹 **8. Eliminación de contenedores e imágenes**

### Eliminar un contenedor específico

```bash
docker rm id_contenedor
```

---

### Eliminar TODOS los contenedores (⚠️ peligroso)

```bash
docker rm $(docker ps -a -q) --force
```

---

### Eliminar una imagen específica

```bash
docker rmi id_imagen
```

---

### Eliminar TODAS las imágenes (⚠️ peligroso)

```bash
docker rmi $(docker images -q)
```

---

## 🌐 **9. Port Forwarding en Docker**

Permite exponer servicios del contenedor al host.

### Sintaxis

```bash
docker run -p PUERTO_HOST:PUERTO_CONTENEDOR imagen
```

Ejemplo HTTP:

```bash
docker run -p 80:8080 mi_imagen
```

Ejemplo UDP:

```bash
docker run -p 53:53/udp mi_imagen
```

---

## 📂 **10. Monturas (volúmenes) y persistencia**

Permiten compartir archivos entre host y contenedor.

### Montar directorio

```bash
docker run -v /home/usuario/datos:/datos mi_imagen
```

### Montar en solo lectura

```bash
docker run -v /home/usuario/datos:/datos:ro mi_imagen
```

Usos reales:

- Persistir bases de datos
    
- Compartir código
    
- Logs
    
- Configuración externa
    

---

## 🧩 **11. Introducción a Docker Compose**

**Docker Compose** permite levantar **múltiples contenedores** usando un solo archivo `docker-compose.yml`.

Ventajas:

- Orquestación sencilla
    
- Servicios interconectados
    
- Ideal para labs vulnerables
    
- Un solo comando para todo
    

Ejemplo conceptual:

```bash
docker-compose up -d
```

Muy usado en:

- VulnHub
    
- HackTheBox
    
- Labs OWASP
    
- Aplicaciones reales (web + db + api)
    


---

## 🎯 **Conclusión**

Docker es una **herramienta fundamental** para:

- Pentesting
    
- Desarrollo
    
- Laboratorios vulnerables
    
- Reproducción de entornos reales
    

Dominar:

- Dockerfile
    
- docker build
    
- docker run
    
- Volúmenes
    
- Port forwarding
    
- Docker Compose
    

te pone **muy por delante del 90%** de la gente.

---
