## 📚**Índice**

- [[#🔥 1. ¿Qué es Drupal y por qué importa en pentesting?]]
- [[#🔍 2. Identificación pasiva de Drupal (fingerprinting sin tocar el servidor)]]
- [[#🧪 3. Identificación activa (detección directa por rutas, archivos y respuestas)]]
- [[#📁 4. Indicadores característicos que revelan Drupal]]
- [[#⚙️ 5. Enumeración automática con Droopescan]]
- [[#🚀 6. Comandos ofensivos manuales para Drupal]]
- [[#🧨 7. Vulnerabilidades comunes y relación con la versión detectada]]
- [[#🐳 8. Laboratorio práctico con CVE-2018-7600 (Drupalgeddon 2)]]
- [[#🛡️ 9. Consideraciones, limitaciones y falsos positivos]]
- [[#📎 10. Recursos adicionales]]
    

---

# 🔥 **1. ¿Qué es Drupal y por qué importa en pentesting?**

**Drupal** es un CMS muy poderoso, flexible y modular. Se usa masivamente en:

- Gobiernos
    
- Universidades
    
- Corporaciones
    
- ONGs
    
- Plataformas de alto tráfico
    

Su arquitectura basada en **módulos**, **temas**, **hooks** y **APIs internas** permite personalización extrema…  
pero también hace que **una instalación mal gestionada se vuelva peligrosa**:

- Módulos vulnerables
    
- Versiones antiguas sin parches
    
- Filtros expuestos
    
- Endpoints accesibles sin autenticación
    

En pentesting, identificar que un sitio es Drupal te abre la puerta a:

- Enumeración profunda (módulos, themes, views)
    
- Detección de versión
    
- Exploits específicos (Drupalgeddon, SQLi, RCE)
    
- Filtros funcionales expuestos al usuario anónimo
    

---

# 🔍 **2. Identificación pasiva de Drupal (fingerprinting sin tocar el servidor)**

Aquí observas _sin realizar requests agresivos_.

## ✔️ Código fuente HTML

Buscar:

```
/sites/default/
/misc/drupal.js
/goto?destination=
/core/misc/
```

Drupal deja huellas en:

- Estructura de URLs
    
- Parámetros específicos
    
- Paths de assets
    

---

## ✔️ Metadatos en HTML

En algunos sitios antiguos:

```
<meta name="Generator" content="Drupal 7 (http://drupal.org)" />
```

---

## ✔️ Patrones de URL característicos

Drupal usa patrones únicos:

- `/?q=admin`
    
- `/?q=user/login`
    
- `/?q=node`
    
- `/node/1`
    
- `/taxonomy/term/`
    
- `/user/register`
    

Si alguno funciona → Drupal confirmado.

---

## ✔️ Verificación con herramientas pasivas

Ejemplo con **WhatWeb**:

```bash
whatweb https://example.com
```

Salida típica:

```
Drupal 7.x detected
PHP 7.x
Apache
```

---

# 🧪 **3. Identificación activa (detección directa por rutas, archivos y respuestas)**

Aquí interactúas con el servidor.

## ✔️ Rutas características de Drupal

Prueba:

```
/CHANGELOG.txt
/README.txt
/install.php
/update.php
/core/CHANGELOG.txt
/modules/
/themes/
/sites/default/
```

### El archivo CLÁSICO:

```
/CHANGELOG.txt
```

→ Muchas instalaciones lo dejan habilitado y revela **versión exacta**.

---

## ✔️ Detalles en la respuesta HTTP

Drupal versiona assets así:

```
/misc/drupal.js
/core/misc/ajax.js?v=8.3.3
```

Si ves `/core/` → Drupal 8+  
Si ves `/misc/` → Drupal 7 o inferior

---

# 📁 **4. Indicadores característicos que revelan Drupal**

## ✔️ Estructura de carpetas

- `/sites/default/settings.php`
    
- `/modules/`
    
- `/themes/`
    
- `/profiles/`
    
- `/core/` (Drupal 8+)
    

## ✔️ Archivos importantes

- `CHANGELOG.txt`
    
- `README.txt`
    
- `settings.php`
    
- `services.yml`
    

## ✔️ Paths de autenticación

- `/user/login`
    
- `/user/password`
    
- `/admin`
    
- `/admin/people`
    

---

# ⚙️ **5. Enumeración automática con Droopescan**

### Repositorio:

```
https://github.com/SamJoan/droopescan
```

Droopescan detecta:

- Versión exacta de Drupal
    
- Módulos instalados
    
- Temas instalados
    
- Vulnerabilidades asociadas
    
- Endpoints expuestos
    

### Uso básico:

```bash
droopescan scan drupal --url https://example.com
```

### Ejemplo de salida típica:

- Drupal 7.56 detected
    
- Module: views
    
- Module: ctools
    
- Vulnerable to CVE-2018-7600
    
- Directory indexing enabled
    

Es extremadamente útil para reconocimiento inicial.

---

# 🚀 **6. Comandos ofensivos manuales para Drupal**

## ✔ Detectar versión (si permiten acceso)

```
/CHANGELOG.txt
/core/CHANGELOG.txt
```

## ✔ Buscar archivos peligrosos

```
/sites/default/settings.php
/sites/default/default.settings.php
```

## ✔ Identificar módulos

```
/modules/
/sites/all/modules/
/sites/default/modules/
```

## ✔ Identificar themes

```
/themes/
/profiles/
/sites/all/themes/
```

---

# 🧨 **7. Vulnerabilidades comunes y relación con la versión detectada**

### ⭐ CVE-2018-7600 (Drupalgeddon 2) — RCE sin autenticación

Afecta:

- Drupal 7.x
    
- Drupal 8.x < 8.3.9
    
- Drupal 8.4.x < 8.4.6
    
- Drupal 8.5.x < 8.5.1
    

La enumeración correcta te dice si las versiones coinciden.

---

### ⭐ Vectores comunes en Drupal:

- Inyección en módulos View
    
- RCE por formularios (Drupalgeddon)
    
- SQLi en módulos contribuidos
    
- Exposición de configuraciones
    
- Filtros XSS en comentarios anon
    
- Módulos admin mal configurados
    

---

# 🐳 **8. Laboratorio práctico con CVE-2018-7600 (Vulhub)**

Repositorio:

```
https://github.com/vulhub/vulhub/tree/master/drupal/CVE-2018-7600
```

Permite practicar:

- Identificación de Drupal
    
- Verificación de versión
    
- Interacción con formulario vulnerable
    
- Explotación de RCE
    
- Conseguir shell (revshell PHP)
    

---

# 🛡️ **9. Consideraciones, limitaciones y falsos positivos**

- Droopescan puede confundir módulos si están protegidos por WAF
    
- Temas personalizados ocultan paths estándar
    
- Reverse proxies pueden esconder versión
    
- Algunos sitios borran CHANGELOG.txt (pero dejan otros rastros)
    
- Caching pesado puede ocultar respuestas dinámicas
    

Siempre confirmar manualmente.

---

# 📎 **10. Recursos adicionales**

- [https://www.drupal.org](https://www.drupal.org/)
    
- Droopescan (OWASP)
    
- HackTricks — Drupal pentesting
    
- WhatWeb / Wappalyzer
    
- Vulhub Labs
    

---

