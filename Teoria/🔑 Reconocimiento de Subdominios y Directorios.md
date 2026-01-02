## 📚 Índice

- [[#🌐 1. Diferencia fundamental: Subdominios vs Directorios]]
- [[#🧭 2. Recon de Subdominios (DNS Recon)]]
    - [[#2.1 Principios y mentalidad]]
    - [[#2.2 Subdominios pasivos (sin tocar el objetivo)]]
    - [[#2.3 Subdominios activos (bruteforce DNS)]]
    - [[#2.4 Descubrimiento profundo (perfil + keywords)]]
    - [[#2.5 Validación de subdominios (Httpx)]]
    - [[#2.6 Expansión (Naabu + Ffuf)]]
- [[#📁 3. Recon de Directorios (Fuzzing HTTP)]]
    - [[#3.1 Qué es fuzzear directorios y qué NO es]]
    - [[#3.2 Recon inicial (rápido)]]
    - [[#3.3 Fuzzing profundo]]
    - [[#3.4 Fuzzing de archivos sensibles]]
    - [[#3.5 Fuzzing de extensiones]]
    - [[#3.6 Detección de parámetros (Param Discovery)]]
- [[#🎯 4. Flujos operativos PRO]]
    - [[#4.1 Flujo Subdominios → Infra → Web]]
    - [[#4.2 Flujo Directorios → LFI / RCE → Pivot]]
- [[#⚡ 5. Plantillas PRO listas para copiar]]
- [[#🧠 6. Reglas mentales que evitan perder tiempo]]
- [[#🚀 7. Tips avanzados Bug Bounty + OSCP]]
- [[#🧪 8. Script Recon Automático (Subdominios + Directorios)]]



---

# 🌐 **1. Diferencia fundamental: Subdominios vs Directorios**

## ✔️ **Subdominio (DNS)**

Es una _máquina/servicio distinto_.  
Está **antes** del dominio:

```
admin.ejemplo.com
api.ejemplo.com
dev.ejemplo.com
```

Se busca con herramientas de DNS (subfinder, amass, assetfinder).

---

## ✔️ **Directorio (HTTP)**

Es una _ruta dentro del servidor web_, después del dominio:

```
ejemplo.com/admin
ejemplo.com/uploads/
ejemplo.com/api/v1/users
```

Se busca con **ffuf**, **dirsearch**, **gobuster**.

---

# 🧭 **2. Recon de Subdominios (DNS Recon)**

## 2.1 **Principios y mentalidad**

El recon de subdominios sirve para encontrar:

- nuevas superficies de ataque
    
- paneles internos expuestos
    
- entornos de desarrollo (dev/stage/test)
    
- APIs ocultas
    
- máquinas completas con CVEs distintas
    

> _Cada subdominio es un nuevo mundo. Trátalo como una máquina distinta._

---

# 2.2 **Subdominios pasivos (sin tocar el objetivo)**

Herramientas que NO generan tráfico directo contra el dominio:

### 🔹 Subfinder

```
subfinder -d ejemplo.com -o subs.txt
```

### 🔹 Assetfinder

```
assetfinder ejemplo.com | tee subs_asset.txt
```

### 🔹 Amass (modo pasivo)

```
amass enum -passive -d ejemplo.com -o subs_amass.txt
```

Combinar resultados:

```
cat subs* | sort -u > subs_all.txt
```

---

# 2.3 **Subdominios activos (bruteforce DNS)**

Aquí SÍ generas consultas DNS.

### 🔹 Amass completo (recomendado)

```
amass enum -active -brute -d ejemplo.com -o subs_brute.txt
```

### 🔹 Fuzzing DNS con Ffuf

```
ffuf -u http://FUZZ.ejemplo.com -w wordlist.txt -fs 0
```

---

# 2.4 **Descubrimiento profundo (perfil + keywords)**

Busca subdominios relacionados por:

- nombres de empleados
    
- tecnología usada
    
- ambiente interno (dev, staging, test, qa)
    

Ejemplo wordlist personalizada:

```
dev
staging
internal
panel
admin
api
legacy
```

---

# 2.5 **Validación de subdominios (Httpx)**

Muchos subdominios existen pero NO responden web.

Httpx valida qué subdominios tienen web real:

```
cat subs_all.txt | httpx -sc -title -tech-detect -o alive.txt
```

Esto te da:

- código HTTP
    
- título
    
- tecnologías
    
- dominios realmente vivos
    

---

# 2.6 **Expansión (Naabu + Ffuf)**

### 🔹 Escanear puertos de subdominios válidos

```
naabu -list alive.txt -top-ports 1000 -o ports_alive.txt
```

### 🔹 Fuzzing de rutas en cada subdominio encontrado

```
ffuf -u https://SUB.DOMINIO/FUZZ -w wordlist.txt -fc 404
```

---

# 📁 **3. Recon de Directorios (Fuzzing HTTP)**

## 3.1 **Qué es fuzzear directorios y qué NO es**

**Sí es:**  
Encontrar **rutas** dentro de un servidor:

```
/admin/
/backup/
/api/v1/
/uploads/
```

**NO es:** descubrir subdominios.

---

## 3.2 **Recon inicial (rápido)**

```
ffuf -u http://IP/FUZZ -w /usr/share/seclists/Discovery/Web-Content/common.txt -fc 404
```

---

## 3.3 **Fuzzing profundo**

```
ffuf -u http://IP/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-large-directories.txt -fc 404
```

---

## 3.4 **Fuzzing de archivos sensibles**

```
ffuf -u http://IP/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-large-files.txt -fc 404
```

---

## 3.5 **Fuzzing de extensiones**

```
ffuf -u http://IP/FUZZ -w wordlist.txt -e .php,.bak,.zip,.old,.txt
```

---

## 3.6 **Detección de parámetros (Param Discovery)**

```
ffuf -u "http://IP/index.php?FUZZ=1" -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt -fw 0
```

---

# 🎯 **4. Flujos operativos PRO**

## 4.1 **Flujo Subdominios → Infra → Web**

1. Enumerar subdominios pasivos
    
2. Combinar y filtrar duplicados
    
3. Validar (httpx)
    
4. Escanear puertos (naabu / nmap)
    
5. Analizar tecnología
    
6. Fuzzing de rutas
    
7. Intentar CVEs + exploits específicos por tecnología
    
8. Pivotar hacia nuevas máquinas internas
    

---

## 4.2 **Flujo Directorios → LFI / RCE → Pivot**

1. Fuzz de directorios
    
2. Fuzz de archivos
    
3. Fuzz de extensiones
    
4. Detectar endpoints PHP / API
    
5. Probar LFI / RFI
    
6. Probar File Upload
    
7. Probar RCE (command injection)
    
8. Si hay shell → enum interna
    

---

# ⚡ **5. Plantillas PRO listas para copiar**

### 🧬 Subdominios (pasivo)

```
subfinder -d dominio.com -o subs1.txt
assetfinder dominio.com > subs2.txt
amass enum -passive -d dominio.com -o subs3.txt
cat subs*.txt | sort -u > subs_all.txt
```

### 🧬 Validación de subdominios

```
cat subs_all.txt | httpx -sc -title -tech-detect -o alive.txt
```

### 🧬 Fuzz directorios

```
ffuf -u https://target.com/FUZZ -w raft-large-directories.txt -fc 404
```

---

# 🧠 **6. Reglas mentales que evitan perder tiempo**

- **Antes del dominio = subdominio. Después del dominio = directorio.**
    
- Si no responde DNS → no lo fuzzes como subdominio.
    
- Si hay un subdominio nuevo → puede ser una máquina entera.
    
- Si un directorio devuelve 403 → fuzzéalo más (muchos 403 esconden joyas).
    
- Si ves `/api/` → fuzz de parámetros obligado.
    
- Si ves `/dev/` → busca backups, `.bak`, `.old`, `.zip`.
    

---

# 🚀 **7. Tips avanzados Bug Bounty + OSCP**

- Los subdominios **dev**, **stage**, **old**, **legacy** son minas de oro.
    
- Fuzzear **subdominios wildcard** requiere filtrar respuestas (fs, fw, fc).
    
- Para sites grandes, usa ffuf con **rate-limit** para no romper nada.
    
- Usa httpx para autodescubrir tecnologías → lanza CVEs precisas.
    

---

# 🧪 **8. Script Recon Automático (Subdominios + Directorios)**

```
#!/bin/bash
domain=$1

echo "[+] Subdominios pasivos"
subfinder -d $domain -o subs1.txt
assetfinder $domain > subs2.txt
amass enum -passive -d $domain -o subs3.txt
cat subs*.txt | sort -u > subs_all.txt

echo "[+] Validando subdominios"
cat subs_all.txt | httpx -sc -title -tech-detect -o alive.txt

echo "[+] Fuzz de directorios"
while read host; do
    ffuf -u https://$host/FUZZ -w /usr/share/seclists/Discovery/Web-Content/common.txt -o "$host-dirs.json"
done < alive.txt
```

---

Si quieres te hago:

- una **versión extendida** con recon DNS + bruteforce avanzados
    
- un módulo de **detección de WAFs**
    
- un flujo **completo para Bug Bounty enterprise**
    

Dime: **"hazme la versión ultra su dominio recon"**.