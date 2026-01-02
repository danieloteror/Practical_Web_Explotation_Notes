## 📚 **Índice**

- [[#🧩 1 Qué es Magento y por qué es tan atacado]]
- [[#🔬 2 Identificación de tecnologías Magento en una página web]]
- [[#🛠️ 3 Magescan — Escáner especializado para Magento]]
- [[#💻 4 Uso real de Magescan (comandos esenciales)]]
- [[#🧪 5 Laboratorio vulnerable Magento SQLi (Vulhub)]]
- [[#💉 6 SQL Injection en Magento (explicación + vector real)]]
- [[#🍪 7 Cookie Hijacking después de SQLi (toma de sesión admin)]]
- [[#🧠 8 Consideraciones de seguridad y vectores adicionales]]

---

# 🧩 **1. Qué es Magento y por qué es tan atacado**

**Magento** es uno de los CMS de comercio electrónico más populares del mundo.  
Usado por tiendas grandes (Nike, Ford, Coca-Cola), maneja:

- Carritos de compra
    
- Usuarios
    
- Pasarelas de pago
    
- Órdenes
    
- Productos
    
- Datos sensibles
    

Esto significa:

👉 **Alto valor para atacantes**  
👉 **Superficie amplia de ataque**  
👉 **Plugins externos con vulnerabilidades**  
👉 **Panel admin crítico (`/admin`)**  
👉 **Dependencia fuerte de la base de datos**

Magento también tiene historiales de:

- Deserialización insegura
    
- SQL Injection
    
- Fugas de sesión
    
- Configuraciones débiles
    
- Rutas administrativas sin ocultar
    

---

# 🔬 **2. Identificación de tecnologías Magento en una página web**

Técnicas comunes para detectar Magento:

### ✔️ Firma en el código fuente

Buscar en `/`:

```html
<meta name="generator" content="Magento"/>
```

### ✔️ Archivos característicos

- `/static/version.../frontend/...`
    
- `/pub/`
    
- `/downloader/`
    
- `/admin` o `/admin_XXXXX`
    

### ✔️ Estructura de URLs de Magento

Ejemplo:

```
/customer/account/login
/checkout/cart
/catalog/product/view
```

### ✔️ Headers HTTP

Magento expone:

```
X-Magento-Tags
X-Frame-Options: SAMEORIGIN
Set-Cookie: mage-cache-sessid
```

### ✔️ Fingerprinting automático (Wappalyzer, WhatWeb)

```bash
whatweb https://target.com
```

---

# 🛠️ **3. Magescan — Escáner especializado para Magento**

**Magescan** es una herramienta hecha específicamente para Magento.  
Detecta:

- Permisos incorrectos
    
- Configuraciones inseguras
    
- Versiones vulnerables
    
- Extensiones peligrosas
    
- Puntos débiles en instalación
    

Repositorio:

➡ [https://github.com/steverobbins/magescan](https://github.com/steverobbins/magescan)

Descargar:

```bash
wget https://raw.githubusercontent.com/steverobbins/magescan/master/magescan.phar
chmod +x magescan.phar
```

---

# 💻 **4. Uso real de Magescan (comandos esenciales)**

Escaneo completo:

```bash
php magescan.phar scan:all https://example.com
```

Escanear versión:

```bash
php magescan.phar scan:version https://example.com
```

Permisos y configuración:

```bash
php magescan.phar scan:file-permissions https://example.com
```

Módulos instalados:

```bash
php magescan.phar scan:modules https://example.com
```

Reportes rápidos:

```bash
php magescan.phar scan:all https://target.com --format=json
```

---

# 🧪 **5. Laboratorio vulnerable de Magento (Vulhub)**

Usaremos:

➡ **Magento 2.2 SQL Injection**  
[https://github.com/vulhub/vulhub/tree/master/magento/2.2-sqli](https://github.com/vulhub/vulhub/tree/master/magento/2.2-sqli)

Se levanta en Docker:

```bash
docker-compose up -d
```

Magento se expone normalmente en:

```
http://localhost:8080
```

---

# 💉 **6. SQL Injection en Magento (explicación + vector real)**

Magento 2.2 tenía una SQLi **pre-auth**, es decir:

👉 El atacante ni siquiera necesita loguearse.

Esto permite:

- Extraer información sensible
    
- Obtener contraseñas hasheadas
    
- Obtener cookies de sesión
    
- Fugar información del admin
    

La vulnerabilidad suele estar en endpoints:

```
/rest/V1/...
/catalogsearch/
```

Con un payload típico:

```sql
' OR 1=1 -- -
```

Para exfiltrar datos específicos:

```sql
UNION SELECT session_id FROM admin_session
```

---

## 🧨 Objetivo del ataque en el laboratorio:

### **Robar la cookie de sesión del administrador**

Esto te permite:

✔ Iniciar sesión sin credenciales  
✔ Acceder al panel `/admin`  
✔ Tomar control de la tienda  
✔ Modificar productos, precios, órdenes  
✔ Insertar skimmers de tarjetas (ataque famoso en Magento)

---

# 🍪 **7. Cookie Hijacking después de SQLi**

Una vez obtienes la cookie:

```
PHPSESSID=xxxxxxxxxxxxxxxxxxxx
```

La envías en el navegador con:

1. EditThisCookie
    
2. DevTools → Application → Cookies
    
3. Request headers (Burp Proxy)
    

Luego:

```
GET /admin
Cookie: PHPSESSID=COOKIE_OBTENIDA
```

Si la sesión está activa:

🔥 **Eres el administrador del e-commerce.**

En un ambiente real, esto equivale a:

- Compromiso total
    
- Robo de datos personales
    
- Cambios de pedidos
    
- Backdoors en plugins
    
- Alterar checkout para robo de tarjetas (MageCart)
    

---

# 🧠 **8. Consideraciones de seguridad y vectores adicionales**

Magento es complejo → también falla de otras formas:

### 🟥 XML External Entity (XXE)

Archivos cargados por el panel.

### 🟥 Deserialización PHP

Frecuente en módulos de terceros.

### 🟥 Install.php accesible

Malas configuraciones permiten reinstalación.

### 🟥 Panel administrativo sin ocultar

URL por defecto: `/admin`.

### 🟥 Configuración insegura de permisos

Muchos fallos detectados por Magescan.

### 🟥 Plugins de pago vulnerables

Stripe, PayPal, Authorize.net → vectores serios.

---

# 🎯 **Conclusión PRO**

Magento es un objetivo:

- Popular
    
- Complejo
    
- Explotable
    
- Usado por empresas grandes
    
- Repleto de módulos vulnerables
    

**Magescan** te permite enumerar y detectar configuraciones débiles.  
**SQL Injection** en Magento = impacto total (control del e-commerce).  
**Cookie Hijacking** → acceso directo al panel administrativo.

Este es un CMS donde un fallo equivale a **pérdidas económicas reales**.

---
