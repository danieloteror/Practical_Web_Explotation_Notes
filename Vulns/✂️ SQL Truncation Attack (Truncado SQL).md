## 📚 **Índice 

- [[#🎯 ¿Qué es el SQL Truncation?]]
- [[#🧠 Root Cause (Causa raíz)]]
- [[#📦 Escenario típico vulnerable]]
- [[#🧪 Ejemplo práctico paso a paso]]
- [[#☠️ Impacto]]
- [[#🕵️ Señales claras de vulnerabilidad]]
- [[#🛠️ Bypass de validación (cliente)]]
- [[#🧬 Excepción SQL silenciosa]]
- [[#🎯 Escenario real (VulnHub)]]
- [[#🧠 Diferencia con SQL Injection]]
- [[#🧱 Mitigación (Defensivo)]]
- [[#🧠 Mentalidad Hacker]]
- [[#📎 Checklist rápida de auditoría]]
- [[#🔥 Conclusión]]

## 🎯 ¿Qué es el SQL Truncation?

El **SQL Truncation Attack** es una técnica que explota **limitaciones de longitud en campos de base de datos** junto con una **validación deficiente**, provocando que el motor SQL **trunque silenciosamente** los datos en lugar de lanzar un error.

👉 El resultado es que una entrada “distinta” para la aplicación termina siendo **idéntica en la base de datos**.

---

## 🧠 Root Cause (Causa raíz)

El ataque ocurre cuando se combinan:

- Campos con **longitud fija** (`VARCHAR(n)`)
    
- **Falta de validación server-side**
    
- **Truncado automático** por el motor SQL
    
- **Eliminación de espacios finales**
    
- Manejo incorrecto de **duplicados**
    

📌 **No es SQL Injection clásica**, es un fallo de **lógica + persistencia**.

---

## 📦 Escenario típico vulnerable

- Formulario de **registro**
    
- Campo `email` o `username`
    
- Límite en DB: `VARCHAR(17)`
    
- Usuario existente:
    
    ```
    admin@admin.com
    ```
    

---

## 🧪 Ejemplo práctico paso a paso

### 1️⃣ Usuario existente en la base de datos

```
admin@admin.com
```

Longitud: **15 caracteres**

---

### 2️⃣ Registro malicioso

Input del atacante:

```
admin@admin.com␠␠a
```

(Longitud total: **18 caracteres**)

✔️ Pasa validación lógica  
✔️ No coincide exactamente con el email existente

---

### 3️⃣ Inserción en la base de datos

La DB trunca a 17 caracteres:

```
admin@admin.com␠␠
```

Luego:

- Los **espacios finales se eliminan**
    
- Resultado final persistido:
    

```
admin@admin.com
```

---

## ☠️ Impacto

🔥 **Account Takeover total**

- Se sobrescribe la contraseña
    
- No se detecta duplicado
    
- El atacante accede como el usuario original
    

---

## 🕵️ Señales claras de vulnerabilidad

🚩 Campos con límites sospechosamente bajos:

- Email ≤ 13 / 15 / 17 caracteres
    
- Username muy corto
    

🚩 Mensajes genéricos:

- “Usuario registrado correctamente”
    
- No hay error de duplicado
    

🚩 Validación **solo en frontend**

---

## 🛠️ Bypass de validación (cliente)

1. Inspeccionar formulario
    
2. Eliminar:
    
    - `maxlength`
        
    - Validaciones JS
        
3. Enviar payload manualmente
    

Ejemplo:

```
user@site.com␠␠x
```

---

## 🧬 Excepción SQL silenciosa

En muchas apps:

- No se lanza error
    
- No hay rollback
    
- Se acepta la operación
    

📌 El truncado **no es tratado como fallo**, sino como comportamiento normal.

---

## 🎯 Escenario real (VulnHub)

**Máquina:** Tornado  
**Plataforma:** VulnHub

- Panel de autenticación
    
- Campo email limitado
    
- Truncado explotable
    
- Reset de contraseña implícito
    

👉 Perfecta para entrenar **SQL Truncation en condiciones reales**

---

## 🧠 Diferencia con SQL Injection

|SQL Injection|SQL Truncation|
|---|---|
|Inyección directa|Lógica de longitud|
|Manipula query|Manipula persistencia|
|Sintaxis maliciosa|Datos “válidos”|
|Muy conocida|Poco detectada|

---

## 🧱 Mitigación (Defensivo)

✅ Validar **longitud real en backend**  
✅ Rechazar truncados, no aceptarlos  
✅ Lanzar error ante duplicados  
✅ Normalizar inputs (trim antes de insertar)  
✅ Usar constraints estrictos (`UNIQUE`)  
✅ Loggear errores de inserción

---

## 🧠 Mentalidad Hacker

- No todo bug es inyección
    
- Los **detalles de longitud importan**
    
- Si algo “corta” en silencio → huele mal
    
- Frontend ≠ seguridad
    

---

## 📎 Checklist rápida de auditoría

-  ¿Campos con límite bajo?
    
-  ¿Validación solo en JS?
    
-  ¿La DB trunca sin error?
    
-  ¿Se eliminan espacios finales?
    
-  ¿Hay UNIQUE mal gestionado?
    
-  ¿Se puede sobrescribir contraseña?
    

---

## 🔥 Conclusión

El **SQL Truncation Attack** demuestra que:

> **La base de datos puede mentirle a la aplicación sin romper nada.**

No rompe queries.  
Rompe **identidades**.

---

