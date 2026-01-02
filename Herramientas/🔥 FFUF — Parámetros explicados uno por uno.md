
## 🟦 **-w** (wordlist)

```
-w archivo.txt
```

**W**ordlist.  
Es el diccionario que FFUF va a usar.

---

## 🟦 **-u** (URL objetivo)

```
-u http://IP/FUZZ
```

**U**RL donde se insertará FUZZ.

---

## 🟦 **FUZZ**

Palabra clave.  
Todo lo que diga “FUZZ” será reemplazado por cada línea del diccionario.

---

## 🟥 **-mc** (Match Code → mostrar SOLO estos códigos)

```
-mc 200
-mc 200,301,403
```

**Match Code** → muestra **solo** las respuestas con esos códigos HTTP.

Ejemplo:  
`-mc 200` significa:  
_solo muéstrame lo que devuelva 200_.

---

## 🟥 **--fc** (Filter Code → ocultar códigos)

```
--fc 404
--fc 400,403
```

**Filter Code** → oculta esos códigos.

Ejemplo:  
`--fc 404` significa:  
_no me muestres nada que devuelva 404_.

---

## 🟧 **-fs** (Filter Size)

```
-fs NUM
```

**Filter Size** → Oculta respuestas cuyo tamaño en **bytes** sea NUM.

Útil cuando TODO devuelve 200 pero con tamaño igual.

---

## 🟧 **-fw** (Filter Words)

```
-fw 20
```

Oculta respuestas con **cantidad de palabras** igual a ese valor.

---

## 🟧 **-fl** (Filter Lines)

```
-fl 12
```

Oculta respuestas con exactamente 12 líneas.

---

## 🟩 **-e** (Extensiones)

```
-e .php,.sh,.cgi
```

FFUF probará:

- FUZZ.php
    
- FUZZ.sh
    
- FUZZ.cgi
    

---

## 🟩 **-X** (Método HTTP)

```
-X POST
-X PUT
```

Igual que en curl → especifica método HTTP.

---

## 🟩 **-d** (Data → cuerpo del POST)

```
-d "user=admin&pass=FUZZ"
```

Para fuzzing de formularios, login, etc.

---

## 🟦 **-H** (Header / Cabeceras)

```
-H "User-Agent: FUZZ"
-H "Host: FUZZ.victim.htb"
```

Sirve para fuzzear VHOSTS o modificar cabeceras.

---

## 🟪 **-t** (threads → velocidad)

```
-t 200
```

Cantidad de **hilos** simultáneos.  
Más hilos = más rápido.  
Demasiados = rompes el server.

---

## 🟫 **-recursion**

```
-recursion -recursion-depth 2
```

Si encuentra directorios, vuelve a fuzzear _dentro de ellos_.

Es potente pero lento.

---

## 🟫 **-o** (output file)

```
-o resultados.json
```

## 🟫 **-of** (output format)

```
-of json
-of html
-of md
```

---

## 🟦 **-v / -vv**

Más verbose. Muestra más detalles.

---

## 🟥 **--sc / --sf / --sl / --sw / --sh**

### MATCH (mostrar solo):

```
--sc 200       ← solo códigos 200
--sl 10        ← solo respuestas con 10 líneas
--sw 300       ← solo 300 palabras
--sh "admin"   ← solo contenido con texto "admin"
```

### FILTER (ocultar):

```
--fc 404       ← oculta esos códigos
--fl 12        ← oculta respuestas de 12 líneas
--fw 100       ← oculta respuestas de 100 palabras
--fs 4242      ← oculta respuestas de 4242 bytes
--fs all       ← oculta respuestas vacías
```

---

# 🔥 EJEMPLOS RÁPIDOS DE PARÁMETROS COMBINADOS

## ✔ Directorios con filtros

```
ffuf -w common.txt -u http://IP/FUZZ -mc 200,301,403
```

## ✔ CGI con extensiones

```
ffuf -w common.txt -u http://IP/cgi-bin/FUZZ -e .sh,.cgi,.pl,.py -mc 200
```

## ✔ Subdominios

```
ffuf -w subdomains.txt -u http://FUZZ.site.com -mc 200
```

## ✔ VHOSTS

```
ffuf -w subdomains.txt -u http://IP/ -H "Host: FUZZ.htb" -mc 200
```

## ✔ POST login

```
ffuf -w rockyou.txt -u http://IP/login -X POST -d "user=admin&pass=FUZZ"
```

---

# 🎯 **TABLA RESUMEN (la que querías)**

|Parámetro|Significado|Qué hace|
|---|---|---|
|`-w`|Wordlist|Diccionario|
|`-u`|URL|Objetivo con FUZZ|
|`-mc`|Match Code|Mostrar solo esos códigos|
|`--fc`|Filter Code|Ocultar esos códigos|
|`-fs`|Filter Size|Ocultar por tamaño (bytes)|
|`-fw`|Filter Words|Ocultar por palabras|
|`-fl`|Filter Lines|Ocultar por líneas|
|`-e`|Extensions|Añade extensiones a FUZZ|
|`-X`|Método HTTP|GET/POST/PUT/etc|
|`-d`|Data POST|Datos POST|
|`-H`|Header|Cabecera personalizada|
|`-t`|Threads|Velocidad|
|`-o`|Output file|Guardar resultados|
|`-of`|Output format|json/csv/html/md|
|`-recursion`|Recursivo|Repetir dentro de directorios|
|`-v`|Verbose|Mostrar más info|
|`--sl/sw/sh/sc`|MATCH|Mostrar solo si coincide|
|`--fl/fw/fs/fc`|FILTER|Ocultar lo que coincide|

---

Bro, si necesitas:

✔ guía completa de Wfuzz  
✔ guía de Gobuster  
✔ comparativa FFUF vs Wfuzz  
✔ cómo detectar falsos positivos  
✔ cómo elegir la wordlist perfecta

Solo dilo. Hay más.