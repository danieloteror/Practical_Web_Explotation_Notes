## 📚 Índice

- [[#🔓 IDOR — Insecure Direct Object References]]
- [[#🧠 Concepto clave (mental model)]]
- [[#📌 ¿Dónde aparecen los IDOR?]]
- [[#🔥 Impacto real]]
- [[#🧪 Detección básica (manual)]]
- [[#🎯 Ejemplo clásico]]
- [[#🧠 Tipos de IDOR]]
- [[#👁️‍🗨️ IDOR Blind]]
- [[#🧰 Enumeración automática]]
- [[#🐍 Script básico de enumeración]]
- [[#🧨 IDOR + lógica rota (combo mortal)]]
- [[#🔐 ¿Por qué ocurre IDOR?]]
- [[#🛡️ Prevención (para defensa / reporte)]]
- [[#🧭 Flujo de trabajo real (pentesting)]]
- [[#🧪 Laboratorios recomendados]]


---

> **IDOR (Insecure Direct Object References)** es una vulnerabilidad de **control de acceso roto** que ocurre cuando una aplicación expone **identificadores internos de objetos** (IDs, UUIDs, nombres, rutas) y **no valida correctamente si el usuario está autorizado** a acceder a ese recurso.

👉 **No es un problema de autenticación, es un problema de autorización.**

---

## 🧠 Concepto clave (mental model)

La app confía en esto:

```
"Si conoces el ID, puedes acceder al recurso"
```

Pero **nunca valida** esto:

```
"¿Este usuario TIENE permiso para este ID?"
```

Resultado: **cualquiera puede acceder a recursos ajenos**.

---

## 📌 ¿Dónde aparecen los IDOR?

### En URLs

```
/orders/123
/profile/45
/api/users/1001
/download?file=invoice_99.pdf
```

### En parámetros

```
user_id=5
account=987
document=2023
```

### En APIs (muy común)

```json
GET /api/orders/124
GET /api/users/2
PATCH /api/profile?id=7
```

---

## 🔥 Impacto real

Un IDOR bien explotado permite:

- 📄 Leer datos privados de otros usuarios
    
- ✏️ Modificar información ajena
    
- ❌ Borrar recursos
    
- 🪪 Tomar control de cuentas
    
- 💸 Fraude (órdenes, pagos, balances)
    
- 🧠 Enumerar toda la base de usuarios
    

👉 **En bug bounty, IDOR = dinero fácil**.

---

## 🧪 Detección básica (manual)

### Paso 1 — Intercepta la request

Con **Burp**, **Postman** o **curl**.

Ejemplo:

```
GET /orders/123
```

### Paso 2 — Cambia el ID

```
GET /orders/124
GET /orders/125
```

### Paso 3 — Observa

- ❌ 403 / 401 → probablemente seguro
    
- ✅ 200 con datos → **IDOR confirmado**
    

---

## 🎯 Ejemplo clásico

Usuario **A**:

```
https://example.com/orders/123
```

Atacante prueba:

```
https://example.com/orders/124
```

Si ve datos del usuario **B** → **IDOR explotado**.

---

## 🧠 Tipos de IDOR

### 🔢 IDOR secuencial

```
1 → 2 → 3 → 4
```

Muy común, muy grave.

---

### 🆔 IDOR con UUID (falso seguro)

```
/users/550e8400-e29b-41d4-a716-446655440000
```

❗ **UUID ≠ control de acceso**  
Si el backend no valida permisos → **sigue siendo IDOR**.

---

### 🔁 IDOR por método HTTP

El frontend bloquea, el backend no:

```
GET /profile/5  ❌
PUT /profile/5  ✅
DELETE /profile/5 ✅
```

---

### 🧪 IDOR en APIs JSON

```json
{
  "user_id": 3,
  "role": "admin"
}
```

Cambias:

```json
{
  "user_id": 1
}
```

---

## 👁️‍🗨️ IDOR Blind

Cuando **no ves respuesta directa**, pero:

- El tamaño cambia
    
- El código HTTP cambia
    
- El tiempo cambia
    

Ejemplo:

```
200 vs 404
```

Suficiente para enumerar.

---

## 🧰 Enumeración automática

### Con Burp Intruder

- Posición: ID
    
- Payloads: `1–1000`
    
- Match: status code / length
    

### Con ffuf

```bash
ffuf -u https://target/api/orders/FUZZ -w ids.txt
```

---

## 🐍 Script básico de enumeración

```python
import requests

for i in range(1, 500):
    r = requests.get(f"https://target/orders/{i}", cookies=auth)
    if r.status_code == 200:
        print(f"[+] ID válido: {i}")
```

---

## 🧨 IDOR + lógica rota (combo mortal)

Ejemplos reales:

- Cambiar `order_id` en **reembolso**
    
- Cambiar `user_id` en **reset de contraseña**
    
- Cambiar `account_id` en **transferencias**
    
- Cambiar `role` indirectamente
    

👉 Aquí es donde **las apps mueren**.

---

## 🔐 ¿Por qué ocurre IDOR?

- ❌ El backend confía en el frontend
    
- ❌ No hay checks de ownership
    
- ❌ Autorización solo en la UI
    
- ❌ APIs mal diseñadas
    
- ❌ “Security by obscurity”
    

---

## 🛡️ Prevención (para defensa / reporte)

- Validar **autorización en el backend**
    
- Verificar **ownership** del recurso
    
- Usar **IDs indirectos + ACL**
    
- Nunca confiar en el cliente
    
- Tests de control de acceso
    

---

## 🧭 Flujo de trabajo real (pentesting)

```
1. Autenticarse como usuario normal
2. Interceptar requests con IDs
3. Cambiar IDs manualmente
4. Probar otros métodos HTTP
5. Enumerar con Intruder / ffuf
6. Buscar impacto (read/write/delete)
7. Documentar con pruebas claras
```

---

## 🧪 Laboratorios recomendados

- **XVWA 1**  
    [https://www.vulnhub.com/entry/xtreme-vulnerable-web-application-xvwa-1,209/](https://www.vulnhub.com/entry/xtreme-vulnerable-web-application-xvwa-1,209/)
    
- **SKF-LABS — IDOR**  
    [https://github.com/blabla1337/skf-labs/tree/master/nodeJs/IDOR](https://github.com/blabla1337/skf-labs/tree/master/nodeJs/IDOR)
    

---
