## 📚 **Índice**

- [[#🎯 **1. Qué es Vim / vi y por qué ES CRÍTICO**]]
- [[#🧠 **2. Filosofía de Vim (esto lo cambia todo)**]]
- [[#🧭 **3. Modos de Vim (core absoluto)**]]
- [[#🚪 **4. Abrir, salir y NO morir en el intento**]]
- [[#🧱 **5. Movimiento eficiente (nivel supervivencia)**]]
- [[#✂️ **6. Edición básica (lo mínimo indispensable)**]]
- [[#📋 **7. Copiar, pegar y borrar (yank / delete)**]]
- [[#🔍 **8. Buscar y reemplazar**]]
- [[#⚙️ **9. Comandos críticos en pentesting / OSCP**]]
- [[#🧪 **10. Vim bajo presión (reverse shells / tty)**]]
- [[#🔥 **11. Errores comunes de novato**]]

---

## 🎯 **1. Qué es Vim / vi y por qué ES CRÍTICO**

**vi** está en **TODOS** los sistemas Unix/Linux.  
**Vim** es vi mejorado.

📌 En pentesting:

- No siempre hay `nano`
    
- No hay VSCode
    
- No hay mouse
    
- A veces **solo hay vi**
    

👉 **Si no sabes Vim, estás incompleto como profesional Linux.**

---

## 🧠 **2. Filosofía de Vim (esto lo cambia todo)**

Vim **no es un editor tradicional**.

👉 No escribes → **das órdenes**  
👉 No usas mouse → **usas verbos**

Ejemplo mental:

> “borra esta palabra”  
> `dw` → _delete word_

💡 Aprendes **lenguaje**, no teclas.

---

## 🧭 **3. Modos de Vim (core absoluto)**

Esto es LO MÁS IMPORTANTE.

|Modo|Qué hace|
|---|---|
|**Normal**|Navegar / comandos|
|**Insert**|Escribir texto|
|**Visual**|Seleccionar|
|**Command**|Comandos `:`|

📌 **Regla de oro**:

> Siempre vuelves a **NORMAL** con `ESC`

---

## 🚪 **4. Abrir, salir y NO morir en el intento**

### Abrir archivo

```bash
vim archivo.txt
```

### Guardar

```vim
:w
```

### Salir

```vim
:q
```

### Guardar y salir

```vim
:wq
```

### Salir sin guardar (emergencia)

```vim
:q!
```

💀 clásico:

> “No puedo salir de Vim”  
> → Falta de `ESC`

---

## 🧱 **5. Movimiento eficiente (nivel supervivencia)**

### Movimiento básico

```text
h  ←
j  ↓
k  ↑
l  →
```

### Movimiento rápido

```vim
w   → siguiente palabra
b   → palabra anterior
0   → inicio de línea
$   → fin de línea
gg  → inicio del archivo
G   → fin del archivo
```

📌 **Esto te ahorra horas.**

---

## ✂️ **6. Edición básica (lo mínimo indispensable)**

### Entrar en modo INSERT

```vim
i   → insertar antes
a   → insertar después
o   → nueva línea debajo
O   → nueva línea arriba
```

### Borrar

```vim
x   → borrar carácter
dw  → borrar palabra
dd  → borrar línea
```

### Deshacer / rehacer

```vim
u       → undo
Ctrl+r  → redo
```

---

## 📋 **7. Copiar, pegar y borrar (yank / delete)**

### Copiar (yank)

```vim
yy   → copiar línea
yw   → copiar palabra
```

### Pegar

```vim
p   → pegar después
P   → pegar antes
```

### Cortar

```vim
dd  → corta línea
```

📌 **Cortar = borrar + copiar**

---

## 🔍 **8. Buscar y reemplazar**

### Buscar

```vim
/palabra
```

- `n` → siguiente
    
- `N` → anterior
    

### Reemplazar (PODER REAL)

```vim
:%s/old/new/g
```

Ejemplo:

```vim
:%s/127.0.0.1/10.10.10.10/g
```

🔥 Clave para pentesting.

---

## ⚙️ **9. Comandos críticos en pentesting / OSCP**

### Editar rápido configs

```bash
vim /etc/hosts
vim .bashrc
vim exploit.py
```

### Editar payloads

- Cambiar IP
    
- Cambiar puerto
    
- Cambiar rutas
    

### Reverse shells

```vim
:%s/4444/9001/g
```

---

## 🧪 **10. Vim bajo presión (reverse shells / tty)**

En shells cutres:

- No hay flechas
    
- No hay Ctrl+C
    
- A veces ni `nano`
    

📌 **Vim sigue funcionando**.

Tip brutal:

```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```

Luego:

```bash
export TERM=xterm
```

👉 Vim usable.

---

## 🔥 **11. Errores comunes de novato**

❌ Quedarse en INSERT  
❌ Usar flechas  
❌ No usar `ESC`  
❌ Editar carácter por carácter  
❌ Pensar que Vim es lento

👉 Vim es lento **solo hasta que lo entiendes**.

---

