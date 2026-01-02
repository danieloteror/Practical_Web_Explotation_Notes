## 📚 **Índice**

- [[#🎯 1. Movimiento rápido del cursor]]
- [[#✂️ 2. Cortar, pegar y borrar texto]]
- [[#🔁 3. Repetición, historial y edición]]
- [[#🧱 4. Autocompletado avanzado]]
- [[#🔥 5. Control de procesos (lo más útil en pentesting)]]
- [[#🧠 6. Atajos poco conocidos pero ultra útiles]]
- [[#🚀 7. Power Moves (nivel hacker)]]
- [[#🛠️ 8. Configurar tus propios atajos (bind)]]

---
# 🎯 **1. Movimiento rápido del cursor**

## 🔹 Ir al inicio de la línea

**`Ctrl + A`**

## 🔹 Ir al final de la línea

**`Ctrl + E`**

## 🔹 Moverse palabra por palabra

- **`Alt + B`** → atrás
    
- **`Alt + F`** → adelante
    

## 🔹 Moverse un carácter

- **`Ctrl + B`** → atrás
    
- **`Ctrl + F`** → adelante
    

---

# ✂️ **2. Cortar, pegar y borrar texto**

## 🔹 Cortar desde el cursor hasta el final

**`Ctrl + K`**

## 🔹 Cortar desde el cursor hasta el inicio

**`Ctrl + U`**

## 🔹 Cortar una palabra hacia atrás

**`Ctrl + W`**

## 🔹 Pegar lo último cortado

**`Ctrl + Y`**

## 🔹 Borrar una palabra hacia adelante

**`Alt + D`**

## 🔹 Borrar un carácter

**`Ctrl + D`**

## 🔹 Borrar hacia atrás (backspace fuerte)

**`Ctrl + H`**

---

# 🔁 **3. Repetición, historial y edición**

## 🔹 Buscar en el historial interactivo

**`Ctrl + R`** y luego escribe parte del comando

## 🔹 Repetir último comando

**`!!`**

## 🔹 Repetir último comando que empieza con “algo”

```
!nmap
!ssh
!cat
```

## 🔹 Traer el último argumento anterior

**`Alt + .`**  
Ejemplo:

```
cp archivo.txt /ruta/larga/
cd Alt+.     → cd /ruta/larga/
```

## 🔹 Editar el comando anterior

**`Ctrl + P`** (anterior)  
**`Ctrl + N`** (siguiente)

---

# 🧱 **4. Autocompletado avanzado**

## 🔹 Autocompletar archivos/comandos

**`Tab`**

## 🔹 Sugerencias múltiples

**`Tab Tab`** muestra todas las opciones.

## 🔹 Expandir `~` rápidamente

```
cd ~ + Tab
```

---

# 🔥 **5. Control de procesos (lo más útil en pentesting)**

## 🔹 Cancelar un proceso

**`Ctrl + C`**

## 🔹 Suspender un proceso

**`Ctrl + Z`**

## 🔹 Reanudar en foreground

```
fg
```

## 🔹 Reanudar en background

```
bg
```

## 🔹 Ver jobs en ejecución

```
jobs
```

---

# 🧠 **6. Atajos poco conocidos pero ultra útiles**

## 🔹 Limpiar pantalla sin usar `clear`

**`Ctrl + L`**

## 🔹 Rehabilitar terminal si se “rompe”

**`reset`**

## 🔹 Expandir variables antes de ejecutar

**`Ctrl + X` + `Ctrl + E`**  
Abre el comando en un editor (nano/vim), luego ejecuta.

---

# 🚀 **7. Power Moves (nivel hacker)**

## 🔹 Volver al último directorio

```
cd -
```

## 🔹 Ver todos los atajos de Bash

```
bind -P
```

## 🔹 Reescribir la línea sin ejecutarla

**`Ctrl + U` → Ctrl + Y → editar**

## 🔹 Autocompletado case-insensitive (hacer permanente)

```
echo "set completion-ignore-case on" >> ~/.inputrc
```

---

# 🛠️ **8. Configurar tus propios atajos (bind)**

Puedes crear atajos personalizados con:

```
bind '"\C-o": "ls -la\n"'
```

Ejemplo: **Ctrl + O** ahora ejecuta `ls -la`.

Hacerlo permanente:

```
echo 'bind "\"\C-o\": \"ls -la\n\""' >> ~/.bashrc
```

---

