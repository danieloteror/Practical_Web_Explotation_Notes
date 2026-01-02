## 📌 **Índice**

- [[#🎯 Qué es Open Redirect y por qué importa]]
- [[#🧠 Modelo mental correcto]]
- [[#🧱 Flujo real de explotación]]
- [[#💣 Casos de uso reales (impacto práctico)]]
- [[#🔍 Identificación y enumeración]]
- [[#🧪 Explotación básica]]
- [[#🧠 Bypass de filtros comunes]]
- [[#🧬 Técnicas avanzadas de evasión]]
- [[#🧪 Laboratorios prácticos (SKF-LABS)]]
- [[#🛡️ Mitigaciones reales (defensivo)]]
- [[#🧠 Open Redirect en bug bounty y pentesting]]

---

## 🎯 **Qué es Open Redirect y por qué importa**

Una vulnerabilidad **Open Redirect** ocurre cuando una aplicación:

- Usa un parámetro para redirigir al usuario
    
- **No valida correctamente el destino**
    
- Permite enviar al usuario a un dominio arbitrario
    

Ejemplo típico:

```
https://victima.com/redirect?url=https://evil.com
```

El dominio legítimo (**victima.com**) ejecuta la redirección.

👉 El navegador confía.  
👉 El usuario confía.  
👉 El atacante abusa.

---

## 🧠 **Modelo mental correcto**

> **El dominio vulnerable no es el destino final.**  
> **Es el trampolín.**

Lo peligroso no es `evil.com`.  
Lo peligroso es que **victima.com** lo envíe.

---

## 🧱 **Flujo real de explotación**

```
[Usuario]
   │ clic
   ▼
https://victima.com/redirect?next=...
   │
   ▼
[Servidor victima.com]
   │
   ▼
https://evil.com/phishing
```

El servidor **legítimo** ejecuta la redirección.

---

## 💣 **Casos de uso reales (impacto práctico)**

### 🎣 Phishing avanzado

- Emails “legítimos”
    
- Dominio real
    
- HTTPS válido
    
- Redirección invisible
    

### 🔓 Bypass de protecciones

- OAuth redirects
    
- SSO flows
    
- Password reset flows
    

### 🧠 Ingeniería social

- “Tu sesión expiró”
    
- “Verifica tu cuenta”
    
- “Accede a la oferta”
    

---

## 🔍 **Identificación y enumeración**

Busca endpoints como:

- `/redirect`
    
- `/login?next=`
    
- `/logout?url=`
    
- `/continue=`
    
- `/return=`
    
- `/callback=`
    

Parámetros sospechosos:

- `url`
    
- `next`
    
- `redirect`
    
- `return`
    
- `continue`
    

---

## 🧪 **Explotación básica**

Ejemplo simple:

```
https://victima.com/redirect?next=https://evil.com
```

Si redirige → **vulnerable**.

---

## 🧠 **Bypass de filtros comunes**

### ❌ Bloqueo de `.` (punto)

Bypass con URL encoding:

```
https://evil%2ecom
```

Doble encoding:

```
https://evil%252ecom
```

---

### ❌ Bloqueo de `https://`

No siempre hace falta:

```
//evil.com
```

El navegador interpreta el protocolo automáticamente.

---

### ❌ Whitelist parcial

Si permiten subdominios:

```
https://victima.com.evil.com
```

---

## 🧬 **Técnicas avanzadas de evasión**

- Doble URL encoding
    
- Mezcla de esquemas (`//`)
    
- Uso de `@`
    

```
https://victima.com@evil.com
```

- Uso de rutas relativas:
    

```
/\\evil.com
```

- Encadenamiento de redirecciones
    

---

## 🧪 **Laboratorios prácticos (SKF-LABS)**

Repositorios:

- Open Redirect 1  
    [https://github.com/blabla1337/skf-labs/tree/master/nodeJs/Url-redirection](https://github.com/blabla1337/skf-labs/tree/master/nodeJs/Url-redirection)
    
- Open Redirect 2  
    [https://github.com/blabla1337/skf-labs/tree/master/nodeJs/Url-redirection-harder](https://github.com/blabla1337/skf-labs/tree/master/nodeJs/Url-redirection-harder)
    
- Open Redirect 3  
    [https://github.com/blabla1337/skf-labs/tree/master/nodeJs/Url-redirection-harder2](https://github.com/blabla1337/skf-labs/tree/master/nodeJs/Url-redirection-harder2)
    

Cada uno introduce:

- Filtros progresivos
    
- Validaciones parciales
    
- Bypasses reales
    

---

## 🛡️ **Mitigaciones reales (defensivo)**

✔️ Whitelist estricta de destinos  
✔️ Usar identificadores internos en lugar de URLs  
✔️ Validar esquema y dominio  
✔️ Normalizar URLs antes de validar  
✔️ Evitar redirecciones dinámicas

Ejemplo seguro:

```js
if (!allowedDomains.includes(parsedUrl.hostname)) {
  reject();
}
```

---

## 🧠 **Open Redirect en bug bounty y pentesting**

- A veces considerado **low**
    
- Escala a **high** si:
    
    - OAuth
        
    - SSO
        
    - Password reset
        
    - Phishing creíble
        

👉 El impacto depende del **contexto**.

---

## 🎯 **Conclusión PRO**

Open Redirect:

- No rompe sistemas
    
- Rompe confianza
    
- Facilita ataques mayores
    

> **No subestimes una redirección.  
> En seguridad, el contexto lo es todo.**

---
