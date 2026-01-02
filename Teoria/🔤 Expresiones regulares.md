## 📚 **Índice**

- [[#🧠 1. Expresiones Regulares (Regex) ]]
    - [[#🔤 1.1 Símbolos fundamentales]]
    - [[#🎯 1.2 Patrones de uso real]]
- [[#🔎 2. `grep` — El Cuchillo Suizo del Pentester]]
    - [[#🧩 2.1 Uso esencial]]
- [[#🧮 3. `awk` — El Lenguaje de las Columnas]]
    - [[#🧩 3.1 Ejemplos esenciales para hacking]]
- [[#🛠️ 4. `sed` — Edición de Texto en Streaming]]
    - [[#🧩 4.1 Reemplazar texto]]
- [[#🔧 5. `tr` — Reemplazar o eliminar caracteres]]
    - [[#🧩 5.1 Ejemplos clave]]
- [[#🔗 6. Pipelines Profesionales — Pequeños Hacks Reales]]
    - [[#🧨 6.1 Ejemplo real]]
    - [[#🔨 6.2 Con curl (muy usado para APIs)]]
- [[#🧠 7. Patrones reales para análisis de logs]]
- [[#🦾 8. Regex + grep + awk + sed — Tácticas avanzadas]]
- [[#📘 9. Resumen de uso correcto]]

# 🧠 **1. Expresiones Regulares (Regex) 

Las expresiones regulares son LO QUE TE PERMITE:

- extraer patrones
    
- detectar anomalías
    
- analizar logs
    
- validar cadenas
    
- automatizar análisis
    

Piensa en ellas como **mini-programas para reconocer texto**.

---

## 🔤 **1.1 Símbolos fundamentales**

|Símbolo|Significado|Ejemplo|
|---|---|---|
|`.`|Un carácter cualquiera|`a.c` → "abc", "a7c"|
|`*`|0 o más repeticiones|`ab*` → "a", "ab", "abbb"|
|`+`|1 o más repeticiones|`ab+` → "ab", "abb"|
|`?`|0 o 1 repetición|`colou?r` → color/colour|
|`^`|Inicio de línea|`^GET`|
|`$`|Final de línea|`404$`|
|`[abc]`|Cualquiera dentro|`[aeiou]`|
|`[^abc]`|Cualquiera excepto|`[^0-9]`|
|`(...)`|Agrupación|`gr(a|
|`|`|OR lógico|
|`\`|Escapar|`\.` busca “.” literal|

---

## 🎯 **1.2 Patrones de uso real**

✔ Emails

```
^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z]{2,}$
```

✔ IPs

```
([0-9]{1,3}\.){3}[0-9]{1,3}
```

✔ Hashes

```
^[a-f0-9]{32}$     # MD5
^[a-f0-9]{40}$     # SHA1
```

✔ URLs

```
https?:\/\/[^\s]+
```

✔ Usuarios con mínimo 8 caracteres

```
^[A-Za-z0-9_]{8,}$
```

---

# 🔎 **2. `grep` — El Cuchillo Suizo del Pentester**

`grep` filtra texto mediante patrones.  
Se usa en análisis de logs, brute-force de patrones, extracción de líneas útiles, etc.

---

## 🧩 **2.1 Uso esencial**

### Buscar "error":

```bash
grep "error" archivo.log
```

### Ignorar mayúsculas/minúsculas:

```bash
grep -i "error" archivo.log
```

### Mostrar líneas que NO contienen:

```bash
grep -v "error" archivo.log
```

### Buscar líneas que **empiezan** con:

```bash
grep "^2024" archivo.log
```

### Buscar usando regex avanzadas:

```bash
grep -E "ERR|WARN|CRIT" archivo.log
```

### Resaltar coincidencias:

```bash
grep --color "failed" auth.log
```

---

# 🧮 **3. `awk` — El Lenguaje de las Columnas**

Si `grep` sirve para buscar,  
**`awk` sirve para ANALIZAR**.

---

## 🧩 **3.1 Ejemplos esenciales para hacking**

### Extraer la segunda columna:

```bash
awk '{print $2}' archivo.txt
```

### Filtrar líneas donde la columna 3 > 100:

```bash
awk '$3 > 100 {print $0}' datos.csv
```

### Usar delimitador por coma:

```bash
awk -F',' '{print $1}' archivo.csv
```

### Sumar una columna:

```bash
awk '{sum+=$1} END {print sum}' archivo.txt
```

### Imprimir columna 1 + columna 10:

```bash
awk '{print $1, $10}' archivo.log
```

---

# 🛠️ **4. `sed` — Edición de Texto en Streaming**

`sed` = _stream editor_.  
Ultra útil para transformación de datos sin abrir un editor.

---

## 🧩 **4.1 Reemplazar texto**

### Reemplazar todas las ocurrencias:

```bash
sed 's/error/fallo/g' archivo.log
```

### Eliminar la quinta línea:

```bash
sed '5d' archivo.txt
```

### Mostrar solo líneas 10 a 15:

```bash
sed -n '10,15p' archivo.txt
```

### Eliminar números al inicio:

```bash
sed 's/^[0-9]*//g' archivo.txt
```

### Eliminar espacios al final:

```bash
sed 's/[ \t]*$//'
```

---

# 🔧 **5. `tr` — Reemplazar o eliminar caracteres**

Muy útil para sanitizar salidas.

---

## 🧩 **5.1 Ejemplos clave**

### Eliminar comillas:

```bash
tr -d '"'
```

### Convertir mayúsculas a minúsculas:

```bash
tr 'A-Z' 'a-z'
```

### Quitar caracteres no imprimibles:

```bash
tr -cd '\11\12\15\40-\176'
```

---

# 🔗 **6. Pipelines Profesionales — Pequeños Hacks Reales**

Combinar herramientas convierte a Bash en un arma.

---

## 🧨 6.1 Ejemplo real

### Filtrar logs → extraer columnas → quitar símbolos

```bash
grep "error" archivo.log \
| awk '{print $1, $3}' \
| sed 's/:/ /g'
```

Pipeline explicado:

- **grep** encuentra el error
    
- **awk** selecciona columnas
    
- **sed** limpia los datos
    

---

## 🔨 6.2 Con curl (muy usado para APIs)

### Tomar campo 1 y 10 de una respuesta:

```bash
curl -s URL | awk '{print $1, $10}'
```

---

# 🧠 **7. Patrones reales para análisis de logs**

### Encontrar todos los 200 OK

```bash
grep " 200 " access.log
```

### Extraer IPs

```bash
grep -oE "([0-9]{1,3}\.){3}[0-9]{1,3}" access.log
```

### Contar IPs únicas

```bash
grep -oE "([0-9]{1,3}\.){3}[0-9]{1,3}" access.log | sort -u | wc -l
```

### La IP con más peticiones

```bash
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head
```

---

# 🦾 **8. Regex + grep + awk + sed — Tácticas avanzadas**

### Extraer usuarios con email corporativo

```bash
grep -E "@empresa\.com$" usuarios.txt
```

### Filtrar por tamaño de payload:

```bash
awk '$10 > 10000' access.log
```

### Reemplazar rutas viejas por nuevas:

```bash
sed 's/\/api\/v1/\/api\/v2/g' routes.txt
```

### Normalizar mayúsculas/minúsculas:

```bash
tr 'A-Z' 'a-z' < archivo.txt
```

---

# 📘 **9. Resumen de uso correcto**

|Herramienta|Útil para|
|---|---|
|**Regex**|Buscar patrones, validar, extraer|
|**grep**|Filtrar líneas|
|**awk**|Procesar columnas, datos estructurados|
|**sed**|Transformar texto|
|**tr**|Eliminar o reemplazar caracteres|
|**Pipelines**|Automatización y análisis rápido|

---

