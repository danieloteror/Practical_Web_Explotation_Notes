## 📚 **Índice

- [[#🚀 1. Filosofía de Recon con Nmap ]]
- [[#🔍 2. Modos de Escaneo (rápido, sigiloso, completo, intrusivo)]]
    - [[#2.1 Escaneo rápido para mapear superficie]]
    - [[#2.2 Escaneo sigiloso (evasión)]]
    - [[#2.3 Escaneo completo/ruidoso (para explotación futura)]]
- [[#⚙️ 3. Escaneos de versiones, servicios y OS detection]]
- [[#🧠 4. Scripts NSE (qué son y cómo usarlos como un profesional)]]
    - [[#4.1 Categorías NSE explicadas]]
    - [[#4.2 Scripts NSE esenciales por objetivo]]
- [[#📦 5. Plantillas de escaneo para uso real]]    
    - [[#5.1 Plantilla Recon Full]]
    - [[#5.2 Plantilla Web Pentest]]
    - [[#5.3 Plantilla Post-exposición (enumeración profunda)]]
- [[#🎯 6. Cheatsheet REAL para Nmap (situacional)]]
- [[#🧩 7. Ejemplos reales de uso en HTB y OSCP]]
- [[#💣 8. Tips avanzados (Firewall bypass, decoys, fragmentación)]]
- [[#🧪 9. Cómo lanzar _conjuntos_ de scripts NSE (profesional)]]
- [[#📁 10. Cómo crear tus propios scripts NSE (plantilla lista)]]

---

# 🚀 **1. Filosofía de Recon con Nmap 

**Un novato escanea.  
Un profesional mapea una superficie de ataque.**

El objetivo no es “ver qué puertos tiene”, sino construir un **modelo mental del sistema**:

1. ¿Qué expone?
    
2. ¿Qué servicios son modificables?
    
3. ¿Qué vectores me abren paso?
    
4. ¿Qué árbol de explotación puedo construir desde aquí?
    

**Mantra PRO:**

> “Primero sé rápido, luego sé sigiloso, luego sé completo.”

---

# 🔍 **2. Modos de Escaneo**

## 2.1 **Escaneo rápido para mapear superficie**

Para obtener los **1000 puertos más comunes**:

```
nmap -Pn -T4 -sS --top-ports 1000 10.10.10.10
```

Para obtener **todos los puertos**, pero rápido:

```
nmap -Pn -T4 -sS -p- 10.10.10.10
```

Optimizado:

```
nmap -Pn -T4 -sS -p- --min-rate 2000 10.10.10.10
```

> **min-rate** fuerza la velocidad mínima: útil en redes lentas pero no blindadas.

---

## 2.2 **Escaneo sigiloso (evasión)**

```
nmap -sS -Pn -T2 --max-retries 1 --defeat-rst-ratelimit 10.10.10.10
```

Modo ultra stealth:

```
nmap -sS -Pn -T1 -f --data-length 50 --randomize-hosts 10.10.10.10
```

Técnicas usadas:

- `-f` ⇒ fragmentación de paquetes
    
- `--data-length` ⇒ bytes extra para joder signatures
    
- `-T1` ⇒ muy lento pero sigiloso
    
- `--randomize-hosts` ⇒ útil en redes grandes
    

---

## 2.3 **Escaneo completo/ruidoso (el necesario para OSCP y HTB)**

Tu escaneo **final antes de explotación**:

```
nmap -A -sC -sV -O -p- 10.10.10.10
```

**Variantes altamente recomendadas:**

```
nmap -sS -sV -sC -A -O --version-all --script-timeout=10m -p- 10.10.10.10
```

---

# ⚙️ **3. Escaneos de versiones, servicios y OS detection**

El combo profesional:

```
nmap -sV --version-all -O --osscan-guess -A 10.10.10.10
```

Cuando solo querés versiones:

```
nmap -sV -p22,80,443,445 10.10.10.10
```

Cuando querés OS sin llamar la atención:

```
nmap -O --osscan-limit --max-retries 2 10.10.10.10
```

---

# 🧠 **4. Scripts NSE (qué son y cómo usarlos como un profesional)**

El corazón de Nmap. Permiten:

- Enumerar servicios complejos
    
- Explotar fallas simples
    
- Bruteforce
    
- Post-exploitation
    
- Info disclosure
    

### 🧩 **4.1 Categorías NSE explicadas**

|Categoría|Uso|
|---|---|
|`default`|Lo básico de Nmap (`-sC`)|
|`safe`|Scripts no intrusivos|
|`vuln`|Detectan vulnerabilidades|
|`auth`|Bruteforce / bypass|
|`discovery`|Enumeración de red|
|`intrusive`|Potencialmente disruptivos|
|`exploit`|Scripts que explotan fallas|

---

### ⚡ **4.2 Scripts NSE esenciales por objetivo**

#### **Objetivos Web**

```
nmap --script http-title,http-headers,http-methods,http-enum,http-vuln* -p80,443 IP
```

#### **SMB**

```
nmap --script smb-os-discovery,smb-enum-shares,smb-enum-users -p445 IP
```

#### **FTP**

```
nmap --script ftp-anon,ftp-syst,ftp-vsftpd-backdoor -p21 IP
```

#### **DNS**

```
nmap --script dns-brute,dns-zone-transfer -p53 IP
```

#### **Vulnerabilidades genéricas**

```
nmap --script vuln IP
```

---

# 📦 **5. Plantillas de escaneo para uso real**

## 5.1 **Plantilla Recon Full (PRO)**

```
nmap -vvv -Pn --min-rate 3000 -p- -sS 10.10.10.10 -oN full_ports.txt
nmap -vvv -Pn -sV -sC -O -p$(cat full_ports.txt | grep open | awk '{print $1}' | sed 's/\/tcp//') 10.10.10.10 -oN detailed_scan.txt
```

## 5.2 **Plantilla Web Pentest**

```
nmap -Pn -p80,443,8080,8443 -sV --script=http-enum,http-title,http-methods,http-headers,http-robots.txt 10.10.10.10
```

## 5.3 **Plantilla Post-exposición**

```
nmap -sV --script vuln -p- 10.10.10.10
```

---

# 🎯 **6. Cheatsheet REAL para Nmap (situacional)**

### Ver qué firewalls hay:

```
nmap --script firewall-bypass -Pn IP
```

### Probar brute-force :

```
nmap --script ssh-brute -p22 IP
```

### Ver banners completos:

```
nmap --script banner -p- IP
```

---

# 🧩 **7. Ejemplos reales de uso (HTB + OSCP)**

### Caso 1: Máquina HTB con SMB expuesto

```
nmap -sS -p445 --script smb-enum-shares,smb-enum-users 10.10.10.10
```

Si ves `READABLE SHARE`, listo: pivot o LFI -> SMB.

### Caso 2: OSCP, web en 80 + 8080

```
nmap -sV --script http-enum,http-vuln* -p80,8080 IP
```

Si encuentras `Tomcat`, prueba:

- Creds por defecto (NSE detecta)
    
- Manager console exploit
    

---

# 💣 **8. Técnicas avanzadas (evasión y bypass)**

### 🔥 Decoys

```
nmap -sS -Pn -D RND:10 IP
```

### 🔥 Spoofing MAC

```
nmap --spoof-mac 00:11:22:33:44:55 -sS IP
```

### 🔥 Fragmentación real

```
nmap -f -sS IP
```

---

# 🧪 **9. Lanzar conjuntos de scripts NSE (PRO)**

### Por categoría

```
nmap --script "vuln or safe" IP
```

### Por múltiples patrones

```
nmap --script "http-* and not http-brute" -p80,443 IP
```

### Por carpetas

```
nmap --script /usr/share/nmap/scripts/ --script-args=unsafe=1 IP
```

### Por lista personalizada (tu arsenal)

Crea un archivo:

`mis_scripts.nse`:

```
http-title
http-enum
vuln
ftp-anon
smb-enum-shares
```

Ejecuta:

```
nmap --script mis_scripts.nse -p- IP
```

---

# 📁 **10. Cómo crear tus propios scripts NSE (plantilla)**

Crea `exploit-mi-vuln.nse`:

```lua
description = [[
Exploit demo para NSE.
]]

author = "Daniel Otero"

license = "Same as Nmap"

categories = {"exploit"}

portrule = function(host, port)
  return port.protocol == "tcp" and port.number == 80
end

action = function(host, port)
  return "Exploit ejecutado contra " .. host.ip
end
```

Correrlo:

```
nmap --script=exploit-mi-vuln.nse IP
```

---

## ¿Quieres que te genere **una versión extendida** con:

- Flujos de reconocimiento para **redes completas**
    
- Cómo automatizar Nmap con Bash
    
- Cómo combinar Nmap + ffuf + gobuster + nikto
    
- Una **nota hermana**: “Recon Total (Web + Infra)”
    

Solo dime **“versión extendida”** y te la creo.