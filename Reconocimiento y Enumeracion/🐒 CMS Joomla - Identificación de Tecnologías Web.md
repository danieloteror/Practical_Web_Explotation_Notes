## 📚 **Índice**

- [[#🔥 1. ¿Qué es Joomla y por qué importa en pentesting?]]
- [[#🧬 2. Identificación de Joomla de forma pasiva (fingerprinting pasivo)]]
- [[#🔍 3. Identificación activa (fingerprinting directo)]]
- [[#🧩 4. Indicadores típicos que revelan Joomla (paths, archivos, headers)]]
- [[#⚙️ 5. Enumeración profunda con Joomscan (OWASP)]]
- [[#🚀 6. Comandos ofensivos manuales útiles para Joomla]]
- [[#🧨 7. Vulnerabilidades comunes + versión detectable]]
- [[#🐳 8. Laboratorio práctico (CVE-2015-8562 – Vulhub)]]
- [[#🛡️ 9. Consideraciones, falsos positivos y buenas prácticas]]
- [[#📎 10. Recursos y documentación útil]]

---

# 🔥 **1. ¿Qué es Joomla y por qué importa en pentesting?**

**Joomla** es un **CMS de código abierto** usado en miles de sitios:

- Gubernamentales
    
- Educativos
    
- ONGs
    
- Empresariales
    
- Blogs avanzados
    
- Intranets
    

Su arquitectura modular (componentes, módulos, plugins, templates) lo hace **altamente extensible**, pero también propenso a vulnerabilidades cuando:

- Plugins están desactualizados
    
- Templates cargan archivos inseguros
    
- Versiones core presentan RCE (como CVE-2015-8562)
    
- Se deja habilitado debug o directorios expuestos
    

Enumerar Joomla correctamente te permitirá:

- Identificar versión
    
- Identificar plugins vulnerables
    
- Detectar paths interesantes
    
- Detectar endpoints expuestos
    
- Localizar vectores para RCE / LFI / SQLi
    

---

# 🧬 **2. Identificación de Joomla de forma pasiva (fingerprinting pasivo)**

La idea: **no alterar el servidor**, observar huellas.

## ✔️ Inspección del código fuente

Buscas patrones típicos:

```
/templates/
/media/system/
/index.php?option=
/components/com_
```

Si ves rutas como:

```
/components/com_content/
/modules/mod_menu/
/plugins/system/
```

→ Joomla confirmado.

---

## ✔️ Archivos característicos en URLs por defecto

Sin romper nada:

```
/administrator/
/readme.txt
/README.txt
/joomla.xml
```

---

## ✔️ Metadatos en HTML

Verifica:

```
<meta name="generator" content="Joomla! - Open Source Content Management" />
```

Muchos admins lo quitan, pero si está → identificación instantánea.

---

## ✔️ Favicon y assets conocidos

Los favicons antiguos de Joomla son reconocibles y pueden compararse con hashes.

---

## ✔️ Wappalyzer / WhatWeb / BuiltWith

Ejemplos:

```bash
whatweb <url>
```

Salida típica:

```
Joomla version X.X detected
PHP 7.x
Apache
```

---

# 🔍 **3. Identificación activa (fingerprinting directo)**

Aquí interactúas con el servidor.

## ✔️ Probar acceso al panel admin

```
/administrator/
```

Si carga un login con el logo Joomla → 100% confirmado.

---

## ✔️ Probar endpoints de componentes

Pruebas típicas:

```
/index.php?option=com_content
/index.php?option=com_users
/index.php?option=com_search
```

Si responden:

→ Joomla.

---

## ✔️ Ver rutas internas accesibles

```
/media/system/js/
/libraries/joomla/
/language/en-GB/
```

---

# 🧩 **4. Indicadores típicos que revelan Joomla**

## 📁 Directorios típicos:

- `/administrator/`
    
- `/components/`
    
- `/modules/`
    
- `/plugins/`
    
- `/templates/`
    

## 📄 Archivos típicos:

- `configuration.php`
    
- `robots.txt` (modificado por Joomla)
    
- `language/en-GB/en-GB.xml`
    

## 🧪 Parámetros típicos:

- `index.php?option=com_XXXX`
    
- `view=XXXX`
    
- `task=XXXX`
    

---

# ⚙️ **5. Enumeración profunda con Joomscan (OWASP)**

**Joomscan** es la herramienta oficial OWASP para auditar Joomla.

Repositorio:

```
https://github.com/OWASP/joomscan
```

### Instalación:

```bash
git clone https://github.com/OWASP/joomscan
cd joomscan
perl joomscan.pl -u <URL>
```

### Uso básico:

```bash
perl joomscan.pl -u https://victima.com
```

### Qué detecta:

- Versión exacta de Joomla
    
- Componentes instalados
    
- Plugins vulnerables
    
- Módulos expuestos
    
- Enumeración de usuarios
    
- Paths inseguros
    
- Archivos backup
    
- Versiones antiguas del núcleo Joomla
    

**⚠️ Importante:**  
Joomscan puede lanzar falsos positivos. No confíes 100% en sus reportes, usa validaciones manuales.

---

# 🚀 **6. Comandos ofensivos manuales útiles para Joomla**

### ✔ Detectar versión (si no está oculto):

```
/administrator/manifests/files/joomla.xml
/media/system/js/core.js
/includes/version.php
```

### ✔ Identificar componentes vulnerables:

```
/components/com_<nombre>/
/index.php?option=com_<nombre>
```

### ✔ Buscar archivos de backup:

```
/configuration.php.bak
/administrator/backups/
/tmp/
/logs/
```

---

# 🧨 **7. Vulnerabilidades comunes + versión detectable**

### ⭐ CVE-2015-8562 — RCE por Object Injection

Laboratorio abajo ↑

Afecta versiones:  
**1.5.x, 2.x, 3.x** (no parcheadas)

Comportamiento típico al detectar Joomla:

- Headers revelan versión de PHP antigua
    
- Joomla expone objetos serializados en headers
    
- Templates personalizados sin sanitización
    

---

### ⭐ Otros vectores comunes:

- **LFI en templates** (inclusión de archivos)
    
- **RFI en extensiones vulnerables**
    
- **SQLi en componentes 3rd party**
    
- **Bruteforce del panel `/administrator/`**
    
- **Enumeración de usuarios via errors**
    
- **Exposición de backup de configuración**
    

---

# 🐳 **8. Laboratorio práctico (CVE-2015-8562 – Vulhub)**

Repositorio Vulhub:

```
https://github.com/vulhub/vulhub/tree/master/joomla/CVE-2015-8562
```

Este laboratorio te permite:

- Enumerar Joomla real
    
- Confirmar versión
    
- Identificar endpoint vulnerable
    
- Explotar Object Injection → RCE
    
- Practicar payloads de explotación
    

---

# 🛡️ **9. Consideraciones, falsos positivos y buenas prácticas**

### ✔ Joomla muy modificado puede ocultar fingerprints

### ✔ Versiones antiguas de Joomla comparten paths

### ✔ Muchos administradores ocultan `/administrator` con plugins

### ✔ Joomscan no detecta todos los componentes

### ✔ Algunas plantillas cambian totalmente la estructura HTML

Siempre valídalo manualmente.

---

# 📎 **10. Recursos y documentación útil**

- Documentación oficial:  
    [https://docs.joomla.org](https://docs.joomla.org/)
    
- OWASP Joomla Vulnerability Scanner (Joomscan)
    
- ExploitDB (plugins vulnerables)
    
- Vulhub Joomla Labs
    
- Wappalyzer / Whatweb
    

---

