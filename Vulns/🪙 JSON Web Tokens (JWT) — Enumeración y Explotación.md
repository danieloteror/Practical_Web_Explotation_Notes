## 📑 Índice

- [[#🔐 JSON Web Tokens (JWT) — Enumeración y Explotación]]
- [[#1. 📘 Qué es un JWT]]
- [[#2. 🧱 Estructura interna de un JWT]]
- [[#3. 📜 RFC 7519 (contexto importante)]]
- [[#4. 🎯 JWT en autenticación y autorización]]
- [[#5. 🔍 Enumeración de JWT]]
    - [[#5.1 Qué significa enumerar un JWT]]
    - [[#5.2 Qué información se busca al enumerar]]
    - [[#5.3 Técnicas comunes de enumeración]]
- [[#6. 🧪 Explotación de JWT]]
    - [[#6.1 Falta de validación de firma]]
    - [[#6.2 Algoritmo "none"]]
    - [[#6.3 Confusión de algoritmos (HS ↔ RS)]]
    - [[#6.4 Secreto débil y fuerza bruta]]
    - [[#6.5 Manipulación de claims]]
- [[#7. 🧰 Herramientas usadas en pruebas de JWT]]
- [[#8. 🧠 Flujo mental del atacante]]
- [[#9. 🛡️ Prevención y buenas prácticas]]
- [[#10. 🔐 Crackeo de JWT con John the Ripper]]
- [[#11. 📌 Resumen ejecutivo]]

---

## 1. 📘 Qué es un JWT

Un **JSON Web Token (JWT)** es un mecanismo estandarizado (**RFC 7519**) para transmitir información entre partes de forma **compacta, autónoma y verificable**.

Se usa principalmente para:

- ✅ autenticación
    
- ✅ autorización
    
- ✅ intercambio seguro de claims
    

⚠️ JWT **NO cifra datos**, solo los codifica y firma.

---

## 2. 🧱 Estructura interna de un JWT

Un JWT tiene tres partes separadas por puntos:

```
HEADER.PAYLOAD.SIGNATURE
```

Cada parte está codificada en **Base64URL**.

### Ejemplo:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
.
eyJ1c2VyIjoiYWRtaW4iLCJyb2xlIjoidXNlciJ9
.
<signature>
```

---

## 3. 📜 RFC 7519 (contexto importante)

El estándar define:

- formato del token
    
- tipos de algoritmos
    
- estructura de claims
    
- validación criptográfica
    

Pero **no obliga** a implementarlo de forma segura → ahí nacen las vulnerabilidades.

---

## 4. 🎯 JWT en autenticación y autorización

### Uso típico:

1. Usuario se autentica
    
2. Servidor genera JWT
    
3. Cliente lo envía en cada request:
    
    ```
    Authorization: Bearer <token>
    ```
    
4. Backend:
    
    - valida firma
        
    - valida expiración
        
    - extrae claims
        
    - autoriza acceso
        

---

## 5. 🔍 Enumeración de JWT

### 5.1 Qué significa enumerar un JWT

Enumerar JWT significa **extraer información útil o inferir debilidades** a partir de:

- estructura
    
- algoritmos
    
- claims
    
- comportamiento del backend
    

No siempre implica romper nada aún.

---

### 5.2 Qué información se busca al enumerar

Desde el JWT puedes obtener:

- algoritmo (`alg`)
    
- tipo (`typ`)
    
- usuario
    
- rol
    
- permisos
    
- timestamps (`iat`, `exp`)
    
- issuer (`iss`)
    
- audience (`aud`)
    
- estructura lógica del backend
    

👉 Todo esto **sin clave secreta**.

---

### 5.3 Técnicas comunes de enumeración

- Decodificar Base64URL
    
- Modificar claims y observar respuesta
    
- Cambiar `alg`
    
- Enviar JWT inválidos
    
- Repetir requests con tokens falsos
    
- Detectar diferencias de error
    
- Ver si acepta tokens sin firma
    

---

## 6. 🧪 Explotación de JWT

---

### 6.1 Falta de validación de firma

Si el backend:

- no valida la firma
    
- o la ignora
    

👉 cualquier token modificado será aceptado.

Ejemplo:

```json
{
  "user": "admin",
  "role": "admin"
}
```

---

### 6.2 Algoritmo `"none"`

Algunos frameworks antiguos aceptaban:

```json
{
  "alg": "none"
}
```

Si el servidor:

- no exige firma
    
- confía en el contenido
    

➡️ autenticación bypass total.

---

### 6.3 Confusión de algoritmos (HS ↔ RS)

Error clásico:

- servidor espera RS256
    
- atacante envía HS256
    
- usa la **clave pública como secreto HMAC**
    

Resultado:  
✔ firma válida  
✔ acceso no autorizado

---

### 6.4 Secreto débil y fuerza bruta

Si el JWT usa **HS256**, necesita un secreto compartido.

Si ese secreto es:

- corto
    
- predecible
    
- palabra común
    

👉 puede romperse por fuerza bruta.

Herramientas suelen probar diccionarios contra la firma hasta validarla.

> ⚠️ Esto solo se practica en laboratorios o sistemas autorizados.

---

### 6.5 Manipulación de claims

Una vez firmado correctamente o sin validación:

```json
{
  "role": "admin",
  "isAdmin": true,
  "uid": 1
}
```

Impactos:

- escalada de privilegios
    
- acceso a endpoints restringidos
    
- bypass de controles
    

---

## 7. 🧰 Herramientas usadas en pruebas de JWT

> (solo con fines educativos / laboratorio)

- 🔧 jwt.io → decodificar / validar
    
- 🧪 Burp Suite → manipulación
    
- 🧰 john → fuerza bruta de secretos débiles
    
- 🧠 scripts propios
    
- 🧪 laboratorios SKF-LABS
    

Repositorio de práctica:  
👉 [https://github.com/blabla1337/skf-labs](https://github.com/blabla1337/skf-labs)

---

## 8. 🧠 Flujo mental del atacante

1. ¿Dónde se usa JWT?
    
2. ¿Qué algoritmo usa?
    
3. ¿Valida firma?
    
4. ¿Acepta `none`?
    
5. ¿Puedo modificar claims?
    
6. ¿El secreto es débil?
    
7. ¿Puedo escalar privilegios?
    

---

## 9. 🛡️ Prevención y buenas prácticas

✅ Usar algoritmos fuertes (`RS256`)  
✅ Validar SIEMPRE firma  
✅ Rechazar `alg: none`  
✅ Rotar claves  
✅ Usar expiraciones cortas  
✅ Validar `iss`, `aud`, `exp`  
✅ No confiar en claims sensibles  
✅ No almacenar secretos débiles  
✅ Monitorear intentos inválidos

---
## 10. 🔐 Crackeo de JWT con John the Ripper 

Para intentar descubrir el **secreto de un JWT**, se debe:

1. Guardar el token completo en un archivo:
    

```bash
jwt.txt
```

2. Ejecutar John indicando el formato HMAC correspondiente:
    

```bash
john jwt.txt --wordlist=/usr/share/wordlists/rockyou.txt --format=HMAC-SHA256
```

📌 **Notas importantes:**

- John solo soporta JWT con algoritmos **HS256, HS384 y HS512**.
    
- El método funciona únicamente si el JWT usa **HMAC (secreto compartido)**.
    
- En este caso, como el algoritmo es **HS256**, el formato correcto es `HMAC-SHA256`.
    

---

## 11. 📌 Resumen ejecutivo

- JWT es solo un **formato**, no seguridad automática
    
- Enumerar JWT permite entender la lógica interna
    
- La explotación surge por:
    
    - mala validación
        
    - secretos débiles
        
    - algoritmos mal usados
        
- JWT mal implementado = **account takeover**
    
- Defensa correcta elimina el 95% de ataques
    

---
