## 📚 Índice

- [[#📌 Qué es]]
- [[#🧠 Mental model]]
- [[#🎯 Impacto real de CSSI]]
- [[#🔍 Detección práctica]]
- [[#🧪 Laboratorio (SKF)]]
- [[#🧩 Casos por contexto]]
- [[#🚨 Cuándo escala a XSS]]
- [[#🧨 Ejemplos prácticos]]
- [[#🧪 Explotación con Burp]]
- [[#🧱 Por qué CSS puede llevar a XSS]]
- [[#🛠️ Mitigaciones]]
- [[#🧾 Template de reporte]]
- [[#🧠 Cheatsheet mental]]

## 📌 Qué es 

**CSS Injection** ocurre cuando tu input termina dentro de CSS (en `<style>`, `style=""`, o un `.css` dinámico) **sin escape contextual**, permitiéndote **inyectar reglas CSS arbitrarias**.

---

## 🧠 Mental model (lo que debes entender sí o sí)

En web, lo importante es **el contexto** donde cae tu input:

- **Contexto CSS**: el parser interpreta reglas CSS.
    
- **Contexto HTML**: el parser interpreta etiquetas.
    
- **Contexto JS**: el motor ejecuta JavaScript.
    

La “magia” de pasar de CSSI a XSS no es por “CSS ejecuta JS” (eso casi nunca), sino por:

### ✅ Escalada típica a XSS

**Si logras salir del contexto CSS y volver a HTML**, puedes inyectar HTML y de ahí a XSS.

Esto suele ocurrir cuando el input cae dentro de:

- un bloque `<style> ... USER ... </style>`
    
- y el backend **no escapa caracteres peligrosos**
    
- y el navegador **termina interpretando HTML después**
    

**Conclusión práctica:**  
CSSI _siempre_ te da control visual.  
CSSI → XSS solo si puedes **romper el contexto** (context break).

---

## 🎯 Qué se puede lograr con CSSI (impacto real, práctico)

### 1) UI Redressing / Phishing visual (muy realista)

- ocultas botones reales
    
- pones overlays
    
- simulas un login
    
- cambias textos (si ya están en el DOM)
    
- engañas al usuario para que haga acciones
    

> Esto puede ser crítico aunque no haya XSS.

### 2) Data exposure indirecto (depende del caso)

- puedes inferir estado del DOM (si existe X elemento)
    
- puedes hacer _side-channels_ (limitado, depende del entorno)
    
- puedes forzar requests a recursos (cargas) — no siempre exfil “directo”
    

### 3) Denial-of-UX

- rompes el layout
    
- haces la página inutilizable
    
- consumes recursos con selectores pesados
    

### 4) Escalada a XSS (solo si el contexto lo permite)

- si puedes salir de `<style>`/atributo `style` hacia HTML
    

---

## 🔍 Detección práctica (cómo encontrar CSSI en un pentest)

### Paso 1: Encuentra puntos donde tu input se refleje

Ejemplos comunes:

- theme/color customization
    
- “firma” o “estado” del usuario
    
- parámetros tipo `color=`, `theme=`, `bg=`, `font=`, `style=`
    
- endpoints `.css` generados dinámicamente:
    
    - `/theme.css?c=...`
        

### Paso 2: Confirma el **contexto exacto**

Abre el response en Burp y busca tu valor:

#### Contexto A — Dentro de `<style>`

```html
<style>
  .profile { color: USER_INPUT; }
</style>
```

#### Contexto B — Dentro de `style=""`

```html
<div style="color: USER_INPUT">Hola</div>
```

#### Contexto C — En CSS servido como archivo

```http
GET /theme.css?color=USER_INPUT
Content-Type: text/css
```

### Paso 3: Confirma ejecución CSS (prueba “benigna”)

Tu primera prueba práctica NO es XSS. Es “¿puedo controlar el CSS?”:

- Cambiar background
    
- Ocultar un elemento
    
- Resaltar elementos
    

**Si puedes ver un cambio visual controlado por tu input → CSSI confirmada.**

---

## 🧪 Laboratorio práctico con SKF 

Repositorio:

- SKF-LABS CSSI: [https://github.com/blabla1337/skf-labs/tree/master/nodeJs/CSSI](https://github.com/blabla1337/skf-labs/tree/master/nodeJs/CSSI)
    

### Flujo de práctica recomendado (sin perder tiempo)

1. Levanta el lab
    
2. Abre Burp y captura la request donde se guarda/reflecta el input
    
3. Identifica el contexto (A/B/C)
    
4. Confirma CSSI con una prueba visual
    
5. Evalúa: ¿solo impacto visual o hay salida de contexto a HTML?
    

---

## 🧩 Casos prácticos por contexto (LO IMPORTANTE)

### ✅ Caso 1 — Input dentro de una propiedad CSS (lo más común)

Ejemplo vulnerable:

```css
.box { color: USER_INPUT; }
```

**Qué pruebas haces:**

- ¿puedo inyectar una segunda regla?
    
- ¿puedo romper la propiedad y crear otra?
    

Resultado típico:

- control de estilos de la página
    
- ocultar elementos
    
- overlays
    

**Impacto práctico:** UI redressing / phishing visual.

---

### ✅ Caso 2 — Input dentro de un selector (más potente)

Ejemplo:

```css
USER_SELECTOR { display: none; }
```

Si controlas selector:

- puedes apuntar a elementos sensibles del DOM
    
- puedes afectar partes específicas (botones de pago, logout, etc.)
    

**Impacto práctico:** manipulación dirigida de la interfaz.

---

### ✅ Caso 3 — Input dentro de `<style>` (el que puede escalar)

Ejemplo vulnerable:

```html
<style>
  body { background: USER_INPUT; }
</style>
```

Aquí te preguntas:

✅ ¿Puedo romper CSS solamente?  
✅ ¿O puedo romper el bloque `<style>` y volver a HTML?

**Si vuelves a HTML**, entonces:

- puedes inyectar HTML
    
- y si inyectas HTML, ya entras en terreno XSS si la app lo permite
    

> **Clave**: no basta “inyectar CSS”; necesitas **salida de contexto**.

---

### ✅ Caso 4 — Inline styles (`style=""`) (limitado pero común)

Ejemplo:

```html
<div style="background: USER_INPUT">
```

Normalmente esto limita a CSS en atributo.  
A veces hay fallos por mal escape de comillas que permiten romper el atributo y pasar a HTML (depende de cómo lo construye el backend/template).

---

## 🚨 Cómo decidir si puede haber XSS (checklist práctico)

### Si tu input cae en `<style> ... </style>`

Preguntas que respondes mirando el response:

- ¿mi input aparece tal cual?
    
- ¿se escapan caracteres tipo `<` `>` `"` `'`?
    
- ¿puedo terminar el bloque CSS de forma prematura?
    
- si logro terminarlo, ¿lo siguiente se interpreta como HTML?
    

### Si tu input cae en `style="..."`

- ¿se escapan comillas?
    
- ¿puedo cerrar el atributo y agregar otro atributo/etiqueta? (depende de templating)
    

**Si no puedes salir del contexto CSS**, no hay XSS:  
hay CSSI (igual puede ser grave).

---

## 🧨 Ejemplos prácticos (seguros) de impacto real SIN XSS

### 1) Ocultar botón de “logout” / “delete”

Objetivo: demostrar riesgo de manipulación de UI (reportable).

- ocultas botones críticos
    
- fuerzas al usuario a clicar un overlay
    

**Qué documentas:**

- captura antes/después
    
- elemento afectado (selector)
    
- flujo de engaño (lo que el usuario termina haciendo)
    

### 2) Overlay de phishing “dentro del sitio”

Objetivo: suplantar UI sin robar credenciales con JS.

- overlay a pantalla completa
    
- texto “sesión expirada”
    
- input fake
    

**Por qué esto importa:**

- ingeniería social + UI redressing
    
- muchas empresas lo consideran alto si hay acciones sensibles
    

---

## 🧪 Metodología de explotación en Burp (práctica real)

### A) Repeater: confirmación rápida

1. capturas request que guarda/refleja input
    
2. mandas en Repeater
    
3. cambias el valor
    
4. recargas la página que renderiza el CSS
    
5. verificas cambio
    

### B) Intruder / Turbo Intruder (cuando el comportamiento es inestable)

Útil si:

- el input se normaliza a veces
    
- hay WAF
    
- hay múltiples rutas de render
    

Usas intruder para barrer variaciones “no peligrosas” y ver cuál rompe contexto (en lab).

---

## 🧱 Por qué “CSS puede convertirse en JS” (la explicación correcta)

CSS **no ejecuta JS mágicamente**.

Lo que pasa en algunos casos es:

- tu input rompe la estructura de `<style>`
    
- el navegador vuelve a parsear como HTML
    
- se inyecta HTML
    
- **y el HTML puede contener JS** (XSS)
    

Así se entiende de forma limpia y real.

---

## 🛠️ Cómo se arregla bien 

### 1) No concatenar strings dentro de CSS

Nunca:

```js
res.send(`body{color:${userInput}}`)
```

### 2) Escape contextual (CSS ≠ HTML)

Si el valor es “color”, usa **whitelist**:

- solo hex
    
- solo rgb()
    
- solo palabras permitidas
    

Ejemplo mental:

- `#RRGGBB`
    
- `rgb(0-255,0-255,0-255)`
    

### 3) Separar datos de estilos

- guarda la preferencia como dato
    
- aplica estilo con clases predefinidas
    
- no generes CSS dinámico con input libre
    

### 4) CSP fuerte (defensa extra)

- bloquear inline scripts
    
- restringir fuentes  
    Esto no arregla la CSSI, pero reduce impacto de una escalada.
    

---

## 🧾 Template de reporte 

**Título:** CSS Injection en parámetro `X` permite UI redressing / posible escalada  
**Impacto:** Alto/Medio según la acción afectada  
**Vector:** Input controlado por usuario se inserta en CSS sin sanitización  
**Evidencia:** capturas antes/después + response mostrando contexto  
**Riesgo:** phishing visual / manipulación de UI / posible salida de contexto (si aplica)  
**Fix:** whitelist + escape contextual + eliminar CSS dinámico

---

## 🧠 Cheatsheet mental (para no perderte)

1. ¿Dónde se refleja mi input?
    
2. ¿En qué contexto exacto cae? (`<style>` / `style=""` / `.css`)
    
3. ¿Puedo controlar CSS de forma visible? (confirmación)
    
4. ¿Puedo salir del contexto CSS hacia HTML? (solo entonces pensar en XSS)
    
5. Si no hay salida: reporta UI redressing / manipulación visual (igual vale)
    

---
