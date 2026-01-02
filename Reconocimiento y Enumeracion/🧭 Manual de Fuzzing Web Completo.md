
# 🕵️‍♂️ **1. ¿Qué es fuzzing web y para qué sirve?**

El fuzzing web consiste en probar miles de rutas, archivos, scripts o subdominios con diccionarios (wordlists) para descubrir:

- Directorios ocultos
    
- Archivos sensibles
    
- APIs privadas
    
- Scripts ejecutables (CGI)
    
- Subdominios (vhosts)
    
- Paneles de administración
    
- Archivos de backup
    
- Funcionalidades ocultas
    

Es una fase crítica en **recon**, especialmente en pentesting web.

---

# 📂 **2. Fuzzing de directorios (lo más común)**

Sirve para encontrar **directorios ocultos** como:

```
/admin
/dev
/include
/uploads
/backup
```

### ✔️ **Herramientas recomendadas**

- Gobuster
    
- Dirb
    
- Wfuzz
    
- Nmap (http-enum)
    

---

## 🔥 **2.1. Gobuster (Directorio estándar)**

```
gobuster dir -w /usr/share/wordlists/dirb/common.txt -u http://example.com/
```

**Wordlists recomendados:**

```
/usr/share/wordlists/dirb/common.txt
/usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
/usr/share/seclists/Discovery/Web-Content/raft-large-directories.txt
```

---

## 🔥 **2.2. Nmap (http-enum)**

```
nmap --script http-enum -p80 10.10.10.10 -vvv
```

Hace fuzzing rápido usando el NSE `http-enum`.

---

## 🔥 **2.3. Dirb (simple pero efectivo)**

```
dirb http://example.com/ /usr/share/dirb/wordlists/common.txt
```

---

# 🗂️ **3. Fuzzing de ARCHIVOS / Scripts CGI (Binarios ejecutables)**

Cuando un sitio usa `/cgi-bin/`, dentro suele haber **scripts ejecutables**:

- `.sh`
    
- `.cgi`
    
- `.pl`
    
- `.py`
    

Esto es _clave_ para vulnerabilidades como **ShellShock**, RCE, etc.

---

## 🔥 **3.1. Dirb con extensiones (IMPORTANTÍSIMO)**

```
dirb http://TARGET/cgi-bin/ /usr/share/dirb/wordlists/common.txt -X .sh,.cgi,.pl,.py
```

Explicación:

- `-X` añade extensiones **a cada palabra del diccionario**
    
- Permite descubrir archivos como:
    
    - `test.sh`
        
    - `admin.cgi`
        
    - `user.pl`
        
    - `status.py`
        

---

## 🔥 **3.2. Wfuzz para CGI**

```
wfuzz -c --hc 404 \
-w /usr/share/seclists/Discovery/Web-Content/raft-small-files.txt \
-u http://TARGET/cgi-bin/FUZZ.sh
```

Cambia `.sh` por:

- `.cgi`
    
- `.pl`
    
- `.py`
    

---

# 🌍 **4. Fuzzing de SUBDOMINIOS (VHOSTS)**

Esto es distinto a directorios. Se usa cuando un servidor responde diferente según el **Host header**.

Ejemplo:

```
Host: admin.example.com
Host: dev.example.com
Host: test.example.com
```

---

## 🔥 **4.1. Wfuzz — Fuzzing de vhosts**

```
wfuzz -c --hc 404 -t 200 \
-w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
-u http://example.com \
-H "Host: FUZZ.example.com"
```

**IMPORTANTE:**  
`-H` SOLO se usa para fuzzear **hosts**, NO directorios.

---

## 🔥 **4.2. Sublist3r**

```
python3 sublist3r.py -d example.com
```

---

## 🔥 **4.3. Activar el subdominio encontrado con /etc/hosts**

```
sudo nano /etc/hosts
```

Agregar:

```
10.129.12.167   admin.example.com
```

Ahora puedes abrirlo en navegador:

```
http://admin.example.com
```

---

# 🧠 **5. WFUZZ – Manual práctico para Obsidian**

### 📌 Comando base

```
wfuzz [opciones] -z payload <URL>
```

`FUZZ` será reemplazado por cada valor del diccionario.

---

## 🔥 **5.1. Fuzzing de Host**

```
wfuzz -c --hc 404 -t 200 \
-w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
-u http://IP \
-H "Host: FUZZ.example.com"
```

---

## 🔥 **5.2. Fuzzing de rutas**

```
wfuzz -c --hc 404 \
-w /usr/share/seclists/Discovery/Web-Content/raft-small-directories.txt \
-u http://IP/FUZZ
```

---

## 🔥 **5.3. Fuzzing de archivos / scripts**

```
wfuzz -c --hc 404 \
-w /usr/share/seclists/Discovery/Web-Content/raft-small-files.txt \
-u http://IP/cgi-bin/FUZZ.sh
```

---

## 🔥 **5.4. Fuzzing con filtrado**

```
--hc 404   → ocultar código 404
--hl N     → ocultar respuestas con N líneas
--hw N     → ocultar respuestas con N palabras
--hh N     → ocultar respuestas con N caracteres
```

Ejemplo:

```
wfuzz --hc 404 --hl 546 -w dic.txt -u http://IP/FUZZ
```

---

## 🔥 **5.5. Añadir extensiones (muy útil)**

```
FUZZ.sh
FUZZ.cgi
FUZZ.pl
FUZZ.py
```

Como:

```
wfuzz -c --hc 404 \
-w wordlist.txt \
-u http://IP/cgi-bin/FUZZ.sh
```

---

# ⚙️ **6. Opciones importantes de Wfuzz (resumen útil)**

- `-c` → colores
    
- `-t N` → hilos concurrentes
    
- `-u URL` → URL objetivo
    
- `-w file.txt` → wordlist
    
- `-H` → cabecera personalizada
    
- `-b` → cookies
    
- `-d` → datos POST
    
- `--hc 404` → ocultar códigos
    
- `--sc` → mostrar solo ciertos códigos
    
- `-X método` → método HTTP
    
- `-p` → proxy
    
- `-R` → fuzzing recursivo
    
- `-e <tipo>` → listar payloads, encoders, etc.
    

---

# 📝 **7. Ejemplos rápidos**

### Directorios:

```
gobuster dir -w common.txt -u http://IP/
```

### Archivos/executables CGI:

```
dirb http://IP/cgi-bin/ common.txt -X .sh,.cgi,.pl,.py
```

### Subdominios:

```
wfuzz -c --hc 404 -w subdomains.txt -u http://IP -H "Host: FUZZ.IP"
```

### Usar vhost encontrado:

```
nano /etc/hosts
```

---

# 🧨 **8. Buenas prácticas para dominar fuzzing web**

- No uses diccionarios gigantes al inicio: empieza con `common.txt` o `raft-small`.
    
- Divide mentalmente:
    
    - subdominios (Host header)
        
    - directorios (FUZZ/)
        
    - archivos (FUZZ.ext)
        
    - parámetros (id=FUZZ)
        
- Fuzzea **/cgi-bin** con EXTENSIONES si buscas scripts.
    
- Revisa SIEMPRE diferencias en:
    
    - status code
        
    - tamaño
        
    - líneas
        
    - palabras
        
- Usa `/etc/hosts` para activar vhosts.
    
- Cuando el servidor devuelva 200 para TODO → **no es ahí**.
    

---

