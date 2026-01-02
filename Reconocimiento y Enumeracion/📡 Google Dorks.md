## 📚 **Índice**

- [[#🔥 1. ¿Qué son los Google Dorks realmente?]]
- [[#🧠 2. ¿Por qué funcionan? (cómo Google indexa contenido sensible)]]
- [[#🧭 3. Categorías de Dorks (clasificación profesional)]]
- [[#🔍 4. Dorks ofensivos esenciales (los que usa un pentester real)]]
- [[#🧪 5. Dorks para enumeración de tecnologías, paneles y CMS]]
- [[#🚨 6. Dorks para encontrar vulnerabilidades explotables]]
- [[#🧩 7. Dorks para fuga de información crítica]]
- [[#🗂️ 8. Dorks para buscar servidores, cámaras, IoT y paneles internos]]
- [[#📁 9. Dorks corporativos (PDF, XLS, leaks, documentación interna)]]
- [[#🎯 10. Cómo automatizar Dorks (OSINT pipelines)]]
- [[#⚡ 11. Cómo usar Dorks junto a otras fases del pentesting]]
- [[#🛡️ 12. Cómo defenderse (para entender aún mejor los ataques)]]

---

# 🔥 **1. ¿Qué son los Google Dorks realmente?**

Los **Google Dorks** son _consultas avanzadas_ en Google que permiten encontrar:

- Información sensible
    
- Archivos expuestos
    
- Paneles de login
    
- Versiones vulnerables de software
    
- Registros de bases de datos
    
- Backups olvidados
    
- Indexación accidental de rutas internas
    

Google dorking = **OSINT ofensivo** → No generas tráfico hacia el servidor objetivo (solo consultas en Google).  
Por eso es perfecto para reconocimiento:

🟢 _Legal si solo consultas Google_  
🔴 _Ilegal si accedes a contenido privado o evades seguridad_

---

# 🧠 **2. ¿Por qué funcionan?**

Porque Google:

- Rastrea **TODO** lo que no esté explícitamente protegido
    
- Indexa directorios mal configurados
    
- Lee metadatos
    
- Crea copias en caché
    
- Indexa archivos PDF, DOCX, XLSX
    
- No distingue qué era “público” o “privado” originalmente
    

Si la empresa la cagó → Google lo encuentra.

---

# 🧭 **3. Categorías de Dorks (clasificación profesional)**

### **1️⃣ Indexación de archivos sensibles**

- .env
    
- .sql
    
- .bak
    
- .git
    
- .log
    

### **2️⃣ Directorios expuestos**

- Index of /
    
- Backups
    
- Uploads
    
- Dev environments
    

### **3️⃣ Paneles de administración**

- Login de WordPress, Joomla, Drupal, Laravel, PHPMyAdmin
    

### **4️⃣ Búsqueda tecnológica**

- Frameworks
    
- Server leaks
    
- Versiones vulnerables
    

### **5️⃣ Información corporativa**

- Documentos internos
    
- PDFs con datos personales
    

### **6️⃣ Infraestructura**

- Cámaras
    
- Routers
    
- IoT
    
- Paneles industriales
    

---

# 🔍 **4. Dorks ofensivos esenciales (los que usa un pentester real)**

### 📁 **Archivos de configuración filtrados**

```
intitle:"index of" ".env"
intitle:"index of" "db_backup"
"DB_PASSWORD" filetype:env
"DB_USER" "DB_PASS"
```

### 🧪 **Bases de datos expuestas**

```
filetype:sql "INSERT INTO"
filetype:db "password"
"backup" filetype:sql
```

### 🔐 **Credenciales y API Keys**

```
"api_key" filetype:json
"Authorization: Bearer" filetype:log
"private_key" filetype:txt
```

### 🗂️ **Repositorios expuestos**

```
intitle:"index of" ".git"
intitle:"index of" ".svn"
```

---

# 🧪 **5. Dorks para enumeración de tecnologías, paneles y CMS**

### 🧩 WordPress

```
inurl:wp-admin
inurl:wp-login.php
"Powered by WordPress"
```

### 🧩 Joomla

```
inurl:administrator/ "joomla"
"index of" "joomla"
```

### 🧩 Drupal

```
inurl:node/
inurl:user/login "drupal"
filetype:txt "Drupal"
```

### ⚙️ Paneles comunes

```
intitle:"login" "admin"
inurl:admin/login
inurl:dashboard "login"
```

---

# 🚨 **6. Dorks para encontrar vulnerabilidades explotables**

### 🐍 PHPMyAdmin expuesto

```
inurl:phpmyadmin
intitle:"phpmyadmin"
```

### 🧊 Jenkins

```
intitle:"Dashboard [Jenkins]"
Jenkins ver. "Login"
```

### 🖧 Apache / Nginx status pages

```
intitle:"Apache Status" "Server Version"
intitle:"nginx" "Welcome to nginx!"
```

### 🧱 Firewalls / WAF deshabilitados

```
intitle:"index of" "mod_security"
```

---

# 🧩 **7. Dorks para fuga de información crítica**

### ☣️ Logs expuestos

```
filetype:log "error" "password"
filetype:log "login"
```

### 🧾 Archivos Excel o PDF con datos internos

```
filetype:xls site:gov "password"
filetype:pdf "confidential"
```

### 🔍 Información personal

```
"Documento Nacional de Identidad" filetype:pdf
"Social Security Number" filetype:pdf
```

---

# 🗂️ **8. Dorks para servidores, cámaras y IoT**

### 📹 Cámaras sin protección

```
intitle:"Live View" inurl:view.shtml
inurl:8080 "axis"
```

### 🏭 Sistemas industriales expuestos

```
intitle:"SCADA"
intitle:"HMI" inurl:8080
```

### 🔌 Routers

```
intitle:"RouterOS" inurl:webfig
inurl:login.cgi "NETGEAR"
```

---

# 📁 **9. Dorks corporativos (PDF, XLS, leaks, documentación interna)**

### 🏢 Manuales internos

```
filetype:pdf "internal use only"
filetype:docx "confidential"
```

### 🗂️ Listas de empleados

```
filetype:xlsx "employee" "email"
```

### 📞 Datos sensibles

```
filetype:csv "phone","email"
```

---

# 🎯 **10. Cómo automatizar Dorks (OSINT pipelines)**

Un pentester profesional no lanza 100 dorks a mano.  
Automatiza:

### 🛠️ Usando **GoogD0rker**

```
python google_dorker.py -d target.com
```

### 🛠️ Usando **GHDB (Google Hacking Database)**

```
https://www.exploit-db.com/google-hacking-database
```

### 🛠️ Scripts propios (bash + curl + HTML parsing)

También puedes usar:

- SearX
    
- Bing Dorking
    
- DuckDuckGo Dorks
    

Para evitar rate limits de Google.

---

# ⚡ **11. Cómo usar Dorks junto a otras fases del pentesting**

### Antes de escanear (fase pasiva):

- Descubres subdominios
    
- Encuentras versiones y paneles
    

### Antes de fuzzing:

- Encuentras rutas sin tocar el servidor
    

### Antes de explotación:

- Puedes hallar archivos de configuración con credenciales
    
- Puedes hallar backups completos del sitio
    

Google es un **fuzzing masivo ya hecho por ti**.

---

# 🛡️ **12. Cómo defenderse (para entender el ataque)**

- Usar `robots.txt` NO es suficiente
    
- Deshabilitar indexing en Apache/Nginx
    
- Evitar poner backups en /public_html
    
- No usar nombres estúpidos como “backup.zip”
    
- Borrar CHANGELOG, README, phpinfo
    
- Revisar periódicamente qué indexa Google con:
    

```
site:empresa.com
```

---

# ⚔️ **Conclusión PRO**

Google Dorking es uno de los pilares de OSINT ofensivo.  
He visto pentesters obtener:

- Credenciales
    
- Backups completos
    
- Acceso a cámaras
    
- Rutas internas
    
- Paneles de admin
    
- Versiones vulnerables exactas
    

**Sin tocar ni un byte del servidor víctima.**

---

