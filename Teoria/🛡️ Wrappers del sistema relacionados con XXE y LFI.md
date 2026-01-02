
## 🔎 Resumen ejecutivo

Los _system wrappers_ (p. ej. `file://`, `php://`, `data://`, `phar://`, `zip://`, `expect://`, etc.) son mecanismos que permiten a un intérprete o runtime tratar distintos recursos (archivos, streams, archivos embebidos) como si fueran ficheros o flujos. En contextos inseguros —entrada no saneada, parsers XML mal configurados, permisos laxos— estos wrappers permiten a un atacante lograr **LFI (Local File Inclusion)**, **RCE** indirecta o **exfiltración de datos** vía **XXE (XML External Entity)**.

Esta nota explica **qué son**, **por qué son peligrosos** en LFI/XXE, **cómo detectarlos** y **cómo mitigar** su uso.

---

## 🧩 ¿Qué son los wrappers del sistema (en este contexto)?

Un _wrapper_ hace que un recurso (archivo, stream, dato embebido) sea accesible mediante una URL o esquema especial. Ejemplos conceptuales:

- `file://` — acceso directo a ficheros del sistema.
    
- `php://` — streams internos de PHP (ej. `php://input`, `php://filter`).
    
- `data://` — datos embebidos (base64, etc.).
    
- `phar://` — acceder a archivos dentro de paquetes PHAR en PHP.
    
- `zip://` — acceso a ficheros dentro de archivos ZIP.
    
- `expect://` — ejecutar comandos (muy peligroso si está disponible).
    

> Muchos handlers y librerías interpretan estos esquemas; si una aplicación concatena rutas con entrada del usuario sin validación, un wrapper puede redirigir esa operación hacia recursos no previstos.

---

## ❗ Por qué son relevantes para **LFI** y **XXE**

### LFI (Local File Inclusion)

- Una inclusión de fichero vulnerable que acepta una ruta controlada por el usuario (`include(user_input)`) puede ser forzada para cargar `file:///etc/passwd`, `php://filter/convert.base64-encode/resource=index.php`, `phar://…` o `zip://…` — desencadenando información sensible o incluso cadenas que facilitan escalada.
    
- Wrappers pueden transformar la salida (p. ej. `php://filter` para base64) o leer recursos no expuestos por la lógica de la app.
    

### XXE (XML External Entity)

- Un parser XML mal configurado que permite entidades externas (`<!ENTITY xxe SYSTEM "file:///etc/passwd">`) permite que el XML pida contenido local o remoto. Aquí los wrappers (p. ej. `file://`, `phar://`) amplían el abanico de recursos accesibles por la entidad.
    
- Similarmente, `data:` URIs o `zip://` pueden ser referencias que el parser resuelve.
    

> En ambos casos la raíz del problema es: **entrada no controlada + capacidades del runtime para resolver esquemas/URIs**.

---

## 🧰 Lista práctica de wrappers a vigilar (no exhaustiva)

> **Enfócate en deshabilitar o limitar el uso de los que están activos en tu stack.**

- `file://`
    
- `php://` (PHP: `php://input`, `php://filter`, `php://temp`, `php://memory`)
    
- `data://`
    
- `zip://`, `phar://`
    
- `expect://` (ejecución directa en algunos entornos)
    
- `gopher://`, `ftp://`, `http://` (cuando `allow_url_fopen`/`allow_url_include` estén activas)
    
- Wrappers específicos de librerías (p. ej. jboss/Java handlers, ZIP filesystem providers)
    

---

## 🧭 Estrategias de mitigación (defensivas, recomendadas)

### A. En el lado de la aplicación (mejor defensa)

1. **Lista blanca de rutas/recursos** — nunca aceptar rutas arbitrarias; exponer sólo recursos explícitos.
    
2. **Normalizar y canonicalizar** rutas con `realpath()` o equivalente y comprobar que están dentro de un directorio permitido (home directory sandbox).
    
3. **Evitar concatenar** entradas de usuario en includes/paths. Preferir mapeos internos (`id => archivo`) en vez de rutas libres.
    
4. **Eliminar funciones peligrosas** o ejecutarlas con mínimos privilegios.
    
5. **Para parsers XML:** deshabilitar entidades externas y DTD. Usar configuraciones seguras del parser (ver abajo por ejemplos).
    
6. **Configuraciones runtime:** desactivar `allow_url_fopen`, `allow_url_include` (PHP) si no son necesarias; restringir wrappers disponibles.
    
7. **open_basedir / container filesystem restrictions** — limitar directorios accesibles por el proceso.
    
8. **Principio de menor privilegio**: procesos con permisos mínimos; evitar que el servidor web corra como root.
    
9. **Validación estric­ta de esquemas/inputs**: usar expresiones regulares y limitaciones de longitud, pero **preferir whitelist**.
    

### B. Configuraciones y hardening por plataforma

#### PHP (recomendaciones)

- `allow_url_fopen = Off`
    
- `allow_url_include = Off`
    
- `open_basedir = /var/www/html/:/tmp/:...` (ajustar según necesidad)
    
- Revocar wrappers no necesarios (si el SAPI/hosting lo permite)
    
- Evitar `include($_GET['file'])`; usar mapeos y `basename()` + comprobación `realpath()`.
    

#### XML parsers (Java, .NET, Python, PHP)

- **Java / XML**: Deshabilitar DTD y external entities:
    
    ```java
    DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
    dbf.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
    dbf.setFeature("http://xml.org/sax/features/external-general-entities", false);
    dbf.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
    dbf.setXIncludeAware(false);
    dbf.setExpandEntityReferences(false);
    ```
    
- **Python / lxml**: usar `defusedxml` o configurar `parser.resolvers` para bloquear URIs externas.
    
- **PHP / libxml**: evitar `libxml_disable_entity_loader` (deprecated) — mejor usar parsers seguros o librerías que deshabiliten DOCTYPE por defecto.
    

### C. Detección y monitoreo

1. **Logging exhaustivo** de accesos a ficheros sensibles (por ejemplo, archivos fuera del árbol de la app).
    
2. **Alertas en SIEM** cuando se detecten accesos a URIs con esquemas no habituales (`php://`, `data:`, `phar://`, etc.).
    
3. **WAF**: reglas que detecten patrones de inclusión de wrappers en parámetros usados para includes/file reads (usar reglas de ejemplo pero evitar false positives).
    
4. **Honeypot files**: ficheros señuelo fuera del árbol habitual y alertas si son leídos.
    
5. **Integridad de ficheros** (FIM) y monitorización de cambios.
    

---

## 🧪 Detección: patrones indicativos (no exploits)

Algunos patrones que habitualmente indican intentos de LFI/abuso de wrappers (para detección, no para exploitation):

- Parámetros que contienen esquemas: `file://`, `php://`, `data:`, `zip://`, `phar://`, `expect://`
    
- Secuencias que intentan normalizar rutas: `../`, `..%2f`, múltiples slash, combinaciones de `\0` (null byte) — estos últimos son más específicos y dependen del entorno.
    
- Uso de `DOCTYPE`, `ENTITY` o inclusión de `SYSTEM`/`PUBLIC` en payloads XML hacia endpoints que procesan XML.
    
- Requests que intentan forzar lectura de rutas como `/etc/passwd`, `/proc/self/environ`, `/var/log/...` desde parámetros de inclusión.
    

Implementa alertas con umbrales para evitar ruido.

---

## ✅ Ejemplos seguros (patrones de código defensivos)

### PHP — validación por whitelist y realpath

```php
// mapeo seguro: no usar la ruta directa del usuario
$pages = [
  'home' => '/var/www/html/pages/home.php',
  'about'=> '/var/www/html/pages/about.php',
];

// entrada desde usuario (ejemplo)
$key = $_GET['page'] ?? 'home';
if (!array_key_exists($key, $pages)) {
  http_response_code(404); exit;
}
include $pages[$key];
```

### PHP — saneamiento de ruta y comprobación de base

```php
$input = $_GET['file'] ?? '';
$base = '/var/www/html/uploads/';
$path = realpath($base . DIRECTORY_SEPARATOR . $input);
if ($path === false || strpos($path, $base) !== 0) {
  // ruta inválida o intenta escapar del directorio
  http_response_code(403); exit;
}
readfile($path); // lectura segura dentro del sandbox
```

### Java — parser XML seguro (configuración)

(ver bloque en sección de mitigación: `disallow-doctype-decl`, desactivar entidades externas).

---

## 🧯 Respuesta ante incidente (si detectas intento/éxito)

1. **Aislar la instancia** afectada (evitar persistencia).
    
2. **Revisar logs** para identificar parámetros y vectores utilizados.
    
3. **Identificar ficheros accedidos** y comprobar si hubo exfiltración.
    
4. **Parchear la vulnerabilidad**: aplicar validación/whitelist+hardening y desplegar.
    
5. **Rotar credenciales** si hay evidencia de exposición.
    
6. **Revisar retrocompatibilidad**: ¿otros servicios usan la misma rutina vulnerable?
    
7. **Documentar y compartir IOA** (Indicadores de Ataque) con equipos de detección.
    

---

## 🔁 Checklist rápido (para pegar en una card o tablero)

-  Desactivar `allow_url_fopen`/`allow_url_include` (si aplica).
    
-  Implementar whitelist de ficheros/IDs.
    
-  Validar rutas con `realpath()` y comprobar prefijo.
    
-  Habilitar open_basedir / contenedor con FS limitado.
    
-  Configurar parser XML seguro (sin DTD ni entidades externas).
    
-  Añadir reglas WAF/SIEM para esquemas inusuales (`php://`, `data:`, `phar://`).
    
-  Monitoreo de accesos a ficheros sensibles y honeypots.
    
-  Revisar permisos de proceso (no ejecutar como root).
    

---

## 📚 Lecturas recomendadas (defensivo)

- OWASP — _XML External Entity (XXE) Prevention Cheat Sheet_
    
- OWASP — _Local File Inclusion (LFI) Prevention Cheat Sheet_
    
- Documentación de tu runtime (PHP `allow_url_*`, Java XMLFactory features, Python `defusedxml`)
    

---

## 🔔 Nota pedagógica final

Los wrappers no son malvados por sí mismos; son poderosas abstracciones que facilitan tareas. El peligro surge cuando **el diseño de entrada/salida asume que la entrada es segura**. Como maestro pedagógico: **trata la entrada como hostil**, limita lo que el runtime puede resolver y da control explícito a la aplicación sobre _qué_ recursos puede abrir.

---

¿Quieres que convierta esto en una **plantilla de Obsidian** con `tag`/`backlinks` adicionales, un template de checklist (con tareas clickables), o que genere un snippet de reglas WAF **conceptuales** para detección? (Lo preparo en la nota para que lo importes directamente).