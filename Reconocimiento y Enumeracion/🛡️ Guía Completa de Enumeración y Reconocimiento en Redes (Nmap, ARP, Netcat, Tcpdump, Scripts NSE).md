
# 🧩 **1. Reconocimiento inicial de la red**

## 🔍 1.1. Ver la IP propia (identificar interfaz)

```
ifconfig
```

O en sistemas modernos:

```
ip a
```

Esto te dice:

- IP interna (ej: 192.168.68.34)
    
- Interfaz (ej: eth0, wlan0)
    
- Máscara y gateway
    

---

## 📡 1.2. Descubrir todos los dispositivos en la red

### 🔥 Escaneo rápido usando ARP (mejor método LAN)

```
sudo arp-scan -I eth0 --localnet
```

ℹ️ _Detecta dispositivos aunque bloqueen ICMP._

---

### 🔥 Nmap para escanear toda la red

Usando la IP base:

Ejemplo:  
Si tu IP es `192.168.68.34`, la red es:  
`192.168.68.0/24`

```
sudo nmap -O -T4 192.168.68.0/24
```

- `-O` → detección de sistema operativo
    
- `-T4` → velocidad alta
    
- `/24` → escanea 256 IPs
    

---

## 🖥️ 1.3. Ver el router/gateway

```
route -n
```

Salida típica:

```
0.0.0.0    192.168.68.1   UG
```

→ Ese es el router.

---

# 🚀 **2. Enumeración profunda del objetivo**

## 🔎 2.1. Escaneo de puertos completo + scripts + versiones

```
sudo nmap -p- -sV -sC --open -sS -vvv -n -Pn 10.10.10.75 -oN escaneo
```

### ✔️ Explicación de parámetros

|Parámetro|Explicación|
|---|---|
|`-p-`|Escanea **todos** los puertos (1–65535).|
|`-sV`|Detecta **versiones** de servicios.|
|`-sC`|Ejecuta scripts **por defecto NSE**.|
|`--open`|Solo muestra puertos **abiertos**.|
|`-sS`|Escaneo SYN (sigiloso, rápido).|
|`-vvv`|Máximo detalle.|
|`-n`|No resuelve DNS → más rápido.|
|`-Pn`|Salta el ping (útil ante firewalls).|
|`-oN escaneo`|Guarda el resultado en archivo.|

---

## 💓 2.2. Ping para probar conectividad

```
ping -c 1 10.0.2.15
```

---


---

# 🔥 **4. Descubrimiento de hosts sigilosos**

## 🟣 4.1. Nmap NULL Scan

```
nmap -sN 192.168.68.0/24
```

Detecta hosts sin enviar flags. Útil para bypassear filtros básicos.

---

# 🛡️ **5. Nmap contra Firewalls**

### ✂️ 5.1. Fragmentación de paquetes

```
nmap -f 192.168.1.10
```

Útil para evadir IDS/IPS antiguos.

---

### 🎭 5.2. Decoys (engañar el origen)

```
nmap -D 8.8.8.8,1.1.1.1,ME 192.168.1.10
```

“ME” representa tu IP real.

---

### 🎧 5.3. Enviar desde puertos controlados

```
nmap --source-port 53 192.168.1.10
```

Muchos firewalls dejan pasar tráfico DNS.

---

### 🧬 5.4. Spoofear dirección MAC

```
nmap --spoof-mac Dell -Pn 192.168.1.10
```

---

### 👻 5.5. Escaneo SYN sigiloso

```
nmap -sS 192.168.1.10
```

---

# 🛰️ **6. Captura de tráfico**

## 📡 6.1. Tcpdump (capturar tráfico a archivo)

```
sudo tcpdump -i eth0 -w captura.cap -v
```

---

## 🔎 6.2. Leer captura con Tshark

```
tshark -r captura.cap 2>/dev/null
```

Filtrar:

```
tshark -r captura.cap -Y "http.request"
```

---

# ⚙️ **7. Scripts NSE (Nmap Scripting Engine)**

## 📁 7.1. Descubrir directorios ocultos (tipo fuzzing)

```
nmap --script http-enum -p80 10.10.10.10 -vvv
```

---

## 🔥 7.2. Buscar vulnerabilidades automáticamente

### Muy intrusivo:

```
nmap --script vuln -p445 10.10.10.10
```

### Menos intrusivo:

```
nmap -p22 10.10.10.10 --script "vuln and safe"
```

---

# 📌 **8. Tips y buenas prácticas**

- Usa `-T4` para velocidad, pero nunca en entornos productivos sin permiso.
    
- Guarda SIEMPRE el resultado con `-oN`.
    
- Si un host bloquea ICMP → usar `-Pn`.
    
- Siempre inicia con escaneo completo (`-p-`).
    
- Usa `arp-scan` antes que `nmap` para descubrir hosts internos.
    

---

# 🛠️ **3. Explotación y Reverse Shells**

## 🐍 3.1. Ejecutar un exploit en Python

```
./exploit.py 192.168.68.54 -payload netcat
```

Notas:

- `./` ejecuta el script local
    
- 192.168.68.54 → víctima
    

---

## 🔁 3.2. Abrir una reverse shell con Netcat

En tu máquina atacante:

```
sudo nc -nlvp 443
```

- `-n` no resuelve DNS
    
- `-l` escucha
    
- `-v` verbose
    
- `-p 443` puerto en escucha
    

---

## 🎭 3.3. Mejorar la terminal tras recibir la shell

```
script /dev/null -c bash
```

Esto te da:

- Prompt decente
    
- Atajos
    
- Sin caracteres raros
    
