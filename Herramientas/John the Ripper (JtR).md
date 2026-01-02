## ✅ ¿Qué es?

**John the Ripper** es una herramienta para:

- auditar contraseñas
    
- comprobar fortaleza de hashes
    
- aprender cómo funcionan los ataques offline
    

Trabaja **rompiendo hashes**, no “hackeando cuentas en vivo”.

---

# 🔹 1. Conceptos clave que debes entender primero

### 🔐 Hash

Un hash es una representación irreversible de una contraseña.

Ejemplos:

- MD5
    
- SHA1
    
- SHA256
    
- bcrypt
    
- NTLM
    
- shadow (Linux)
    
- ZIP / RAR / PDF hashes
    

John **no rompe contraseñas directamente**, rompe **hashes**.

---

### 📂 Flujo básico de trabajo

1. Obtener hash (de un laboratorio o archivo)
    
2. Identificar tipo de hash
    
3. Ejecutar John
    
4. Usar wordlists / reglas / fuerza bruta
    
5. Ver resultados
    

---

# 🔹 2. Instalación

### Kali / Parrot

```bash
sudo apt install john
```

### Versión recomendada

Siempre usa **Jumbo** (viene por defecto en Kali):

```bash
john --list=formats
```

---

# 🔹 3. Identificar el tipo de hash

Antes de atacar, **identifica el hash**:

```bash
hashid hash.txt
```

o

```bash
john --list=formats | grep -i md5
```

Ejemplo hash:

```
5f4dcc3b5aa765d61d8327deb882cf99
```

→ MD5

---

# 🔹 4. Ataque básico (modo automático)

```bash
john hash.txt
```

John intentará:

- reglas básicas
    
- wordlist por defecto
    
- heurísticas
    

---

# 🔹 5. Usar wordlists (lo más común)

### Wordlist clásica

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

👉 RockYou es la base de casi todo.

---

# 🔹 6. Ver contraseñas crackeadas

```bash
john --show hash.txt
```

Salida típica:

```
user:password123
```

---

# 🔹 7. Modos de ataque importantes

## 🔸 Wordlist + reglas (muy potente)

```bash
john --wordlist=rockyou.txt --rules hash.txt
```

Las reglas hacen cosas como:

- password → Password
    
- password → password123
    
- password → P@ssw0rd
    
- invertir
    
- añadir números
    
- mayúsculas
    

📌 80% de cracks reales salen de aquí.

---

## 🔸 Ataque incremental (fuerza bruta inteligente)

```bash
john --incremental hash.txt
```

Puedes limitarlo:

```bash
john --incremental=Digits
john --incremental=Lower
```

---

## 🔸 Ataque por máscara

Cuando conoces el patrón:

Ejemplo:

- 8 caracteres
    
- empieza por mayúscula
    
- termina en número
    

```bash
john --mask='?u?l?l?l?l?l?l?d' hash.txt
```

### Máscaras útiles:

|Símbolo|Significado|
|---|---|
|?l|letra minúscula|
|?u|mayúscula|
|?d|dígito|
|?s|símbolo|
|?a|todo|

---

# 🔹 8. Ataques a hashes comunes

## Linux shadow

```bash
unshadow passwd shadow > hashes.txt
john hashes.txt
```

---

## NTLM (Windows)

```bash
john --format=NT hash.txt
```

---

## ZIP

```bash
zip2john file.zip > zip.hash
john zip.hash
```

---

## RAR

```bash
rar2john file.rar > rar.hash
john rar.hash
```

---

## PDF

```bash
pdf2john file.pdf > pdf.hash
john pdf.hash
```

---

# 🔹 9. Usar sesiones (reanudar ataques)

```bash
john --session=lab1 hash.txt
```

Reanudar:

```bash
john --restore=lab1
```

---

# 🔹 10. Ver estado en tiempo real

```bash
john --status
```

---

# 🔹 11. Archivo de configuración (avanzado)

Ubicación:

```
/etc/john/john.conf
```

Ahí puedes:

- crear reglas personalizadas
    
- modificar modos
    
- optimizar ataques
    

Ejemplo simple de regla:

```
[List.Rules:MiRegla]
Az"123"
```

---

# 🔹 12. Buenas prácticas reales (importante)

✅ Usa John solo cuando:

- el sistema es tuyo
    
- tienes autorización explícita
    
- estás en laboratorios (HTB, TryHackMe, VulnHub)
    

❌ Nunca contra sistemas reales sin permiso.

---

# 🔹 13. Flujo profesional recomendado (pentesting)

1. Identificar hash
    
2. Clasificar tipo
    
3. Probar wordlist + reglas
    
4. Probar máscaras inteligentes
    
5. Guardar resultados
    
6. Documentar
    

---

# 🔹 14. Comando resumen rápido

```bash
john hash.txt
john --wordlist=rockyou.txt hash.txt
john --rules hash.txt
john --mask='?l?l?l?l?d?d' hash.txt
john --show hash.txt
```

---

### 🔐 Crackeo de JWT con John the Ripper (HS256)

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

Si quieres, puedo convertir esto en **bloque Obsidian con índice**, o integrarlo dentro de tu nota grande de _JWT / Auth attacks_ con ejemplos reales tipo HTB.