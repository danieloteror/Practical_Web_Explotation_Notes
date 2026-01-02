## 📚 **Índice

- [[#🌐 1. Filosofía del Recon Total]]
- [[#📡 2. Infra Recon (Nmap + capas de red)]]
    - [[#2.1 Descubrimiento de hosts]]
    - [[#2.2 Mapeo completo de puertos (rápido y luego profundo)]]
    - [[#2.3 Detección de servicios y versiones]]
    - [[#2.4 Enumeración específica por servicio (SMB, FTP, SSH, etc.)]]
- [[#🔎 3. Recon Web (detección, mapeo y fingerprinting)]]
    - [[#3.1 Fingerprinting inicial]]
    - [[#3.2 Enumeración de rutas y fuzzing profundo]]
    - [[#3.3 Fingerprint tecnológico y CVEs relevantes]]
    - [[#3.4 Recon de archivos sensibles]]
    - [[#3.5 Enumeración SSL/TLS]]
- [[#🛠️ 5. Plantillas PRO listas para copiar]]
    - [[#5.1 Recon Completo Infra (2 fases)]]
    - [[#5.2 Recon Completo Web (3 fases)]]
    - [[#5.3 Combo Maestro (Infra + Web)]]
- [[#⚡ 6. Mini-flowcharts (cómo pensar en segundos)]]
- [[#📁 7. Wordlists indispensables para Recon]]
- [[#🧪 8. Recon programático (bash, python, automatización real)]]
- [[#🚀 9. Tips de alto nivel (OSCP, HTB, BB)]]

---

# 🌐 **1. Filosofía del Recon Total**

Recon no es “escanea todo”.  
Recon es **descubrir TODO lo que existe** para luego **priorizar vectores de ataque reales**.

Tu objetivo:

1. Ver _qué está expuesto_.
    
2. Ver _qué tecnología corre_.
    
3. Entender _dónde es débil_.
    
4. Convertir **información → explotación**.
    

> _Un pentester mediocre empieza con nmap.  
> Un pentester profesional empieza con una hipótesis:  
> “¿Dónde rompo primero?”_

---

# 📡 **2. Infra Recon (Nmap + capas de red)**

## 2.1 **Descubrimiento de hosts**

Rango completo rápido:

```
nmap -sn 10.10.10.0/24 -oN hosts.txt
```

Sin ping (firewalled):

```
nmap -Pn -sn 10.10.10.0/24 -oN hosts.txt
```

ARP (LAN):

```
arp-scan -l
```


---
## 2.2 **Mapeo completo de puertos (rápido y luego profundo)**

### 🔹 **Fase 1: Rápida — solo descubrir puertos abiertos**

Guarda el escaneo rápido en un archivo:

```
nmap -Pn -sS --min-rate 3000 -p- IP -oN ports_fast.txt
```

### 🔹 **Fase 2: Profunda — detectar versiones + scripts NSE**

1. **Extraer automáticamente los puertos abiertos del archivo anterior**  
    (convierte `22/tcp`, `80/tcp`, etc. en `22,80,...`):
    
```
ports=$(grep '/tcp' ports_fast.txt | cut -d'/' -f1 | tr '\n' ',')
```

2. **Escanear solo esos puertos, pero ahora en modo profundo**:
    
```
nmap -Pn -sV -sC -O -p$ports IP -oN ports_detailed.txt
```


---

## 2.3 **Detección de servicios y versiones**

Ya con puertos conocidos:

```
nmap -sV --version-all -p$ports IP
```

---

## 2.4 **Enumeración específica por servicio**

### SMB

```
nmap --script smb-enum-shares,smb-enum-users,smb-os-discovery -p445 IP
```

### FTP

```
nmap --script ftp-anon,ftp-syst -p21 IP
```

### SSH

```
nmap --script ssh-auth-methods,ssh-hostkey -p22 IP
```

### DNS

```
nmap --script dns-brute,dns-zone-transfer -p53 IP
```

### MySQL

```
nmap --script mysql-enum,mysql-users -p3306 IP
```

---

# 🔎 **3. Recon Web (detección, mapeo, fingerprinting)**

## 3.1 **Fingerprinting inicial**

```
whatweb http://IP
```

Trae:

- Tecnologías
    
- CMS
    
- Frameworks
    
- Headers
    
- Versión del servidor
    

También:

```
nmap -p80,443 --script http-title,http-server-header,http-methods IP
```

---

## 3.2 **Enumeración de rutas y fuzzing**

### Rutas comunes

```
ffuf -u http://IP/FUZZ -w /usr/share/seclists/Discovery/Web-Content/common.txt
```

### Profundo

```
ffuf -u http://IP/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-large-directories.txt
```

### Archivos

```
ffuf -u http://IP/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-large-files.txt
```

### Extensiones

```
ffuf -u http://IP/FUZZ -w wordlist.txt -e .php,.bak,.old,.txt,.zip
```

### Fuzz de parámetros (PARAMETER DISCOVERY)

```
ffuf -u "http://IP/page.php?FUZZ=test" -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt
```

---

## 3.3 **Fingerprint tecnológico + CVEs**

Con WhatWeb y Wappalyzer identificado:

Busca CVEs:

```
searchsploit tecnología versión
```

Ejemplo:

```
searchsploit apache 2.4.49
```

---

## 3.4 **Recon de archivos sensibles**

```
nmap -p80 --script http-robots.txt,http-config-backup,http-backup-finder IP
```

También con ffuf:

```
ffuf -u http://IP/FUZZ -w /usr/share/seclists/Discovery/Web-Content/sensitive-files.txt
```

---

## 3.5 **Enumeración SSL/TLS**

```
nmap --script ssl-cert,ssl-enum-ciphers -p443 IP
```

Con `testssl` si quieres full info:

```
testssl.sh https://IP
```

---

# 🛠️ **5. Plantillas PRO listas para copiar**

## 5.1 **Recon Completo Infra (2 fases)**

### Fase 1 — Descubrimiento + Puertos

```
nmap -Pn -sS -p- --min-rate 3000 IP -oN quick_ports.txt
```

### Fase 2 — Profundo

```
ports=$(cat quick_ports.txt | grep 'open' | cut -d'/' -f1 | tr '\n' ',')

nmap -Pn -sV -sC -O -p$ports IP -oN infra_full.txt
```

---

## 5.2 **Recon Completo Web (3 fases)**

### Fase 1 — Fingerprinting

```
whatweb http://IP
```

```
nmap -p80,443 --script http-enum,http-methods,http-title,http-headers IP
```

### Fase 2 — Directory Fuzz

```
ffuf -u http://IP/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-large-directories.txt
```

### Fase 3 — Archivos sensibles + parámetros

```
ffuf -u http://IP/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-large-files.txt
```

```
ffuf -u "http://IP/page.php?FUZZ=x" -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt
```

---

## 5.3 **Combo Maestro (Infra + Web)**

(_lo que usarás en 90% de tus máquinas_)

```
nmap -Pn -sS -p- --min-rate 3000 IP -oN ports.txt
whatweb http://IP | tee web_fingerprint.txt
nmap -p80,443 --script http-title,http-server-header,http-methods IP -oN web_nse.txt
```

Luego:

```
ffuf -u http://IP/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-large-directories.txt -o ffuf_dirs.json
```

---

# ⚡ **6. Mini-flowcharts (cómo pensar en segundos)**

### 🔥 Si un puerto expuesto es **HTTP**:

1. Title
    
2. Headers
    
3. Whatweb
    
4. Fuzz
    
5. Archivos
    
6. Parámetros
    
7. CVEs
    

### 🔥 Si un puerto expuesto es **SMB**:

1. Shares
    
2. Users
    
3. Credenciales
    
4. Archivos → panel web
    
5. Pass-the-hash → SSH / RDP
    

### 🔥 Si encuentras **FTP anónimo**:

1. Descarga todo.
    
2. Busca `/www/`.
    
3. Prueba upload → Web Shell.
    

---

# 📁 **7. Wordlists indispensables**

- `raft-large-directories.txt`
    
- `raft-large-files.txt`
    
- `common.txt`
    
- `extensions_common.txt`
    
- `big.txt`
    
- `burp-parameter-names.txt`
    
- `sensitive-files.txt`
    

---

# 🧪 **8. Recon programático (automatización real)**

## Script Bash PRO

```
#!/bin/bash

IP=$1

echo "[+] Escaneo rápido"
nmap -Pn -sS -p- --min-rate 3000 $IP -oN ports.txt

ports=$(grep 'open' ports.txt | cut -d'/' -f1 | tr '\n' ',')

echo "[+] Escaneo profundo"
nmap -Pn -sV -sC -O -p$ports $IP -oN infra.txt

echo "[+] Web fingerprint"
whatweb http://$IP > web.txt

echo "[+] Fuzz de directorios"
ffuf -u http://$IP/FUZZ -w /usr/share/seclists/Discovery/Web-Content/common.txt -o fuzz.json
```

---

# 🚀 **9. Tips de alto nivel (OSCP, HTB, Bug Bounty)**

- Si ves **puerto raro (8080, 8443, 5000, 8000)** → _seguro hay web_.
    
- Si ves **SMB** → invierte tiempo: casi siempre hay un vector.
    
- Si ves **UDP** → piensa en SNMP (puede darte TODO).
    
- Si ves **archivo .bak** → jackpot.
    
- Si ves **panel admin** → prueba credenciales usadas en SMB/FTP/DB.
    
- Si ves **WordPress** → siempre prueba `/wp-admin/`, `/xmlrpc.php`.
    

---
