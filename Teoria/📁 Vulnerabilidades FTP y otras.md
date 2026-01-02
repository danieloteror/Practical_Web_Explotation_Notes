## 📚 **Índice**

- [[#🎯 **1. Introducción a FTP y su superficie de ataque**]]
- [[#⚠️ **2. Anonymous FTP login allowed**]]
- [[#🔍 **3. Enumeración de versión FTP**]]
- [[#🛠 **4. Acceso y enumeración manual por FTP**]]
- [[#🚀 **5. Ataques de fuerza bruta con Hydra**]]
- [[#🧠 **6. Credenciales filtradas y OSINT (DeHashed)**]]
- [[#🧪 **7. Escenario típico en CTF / OSCP**]]
- [[#🧱 **8. Mitigaciones y buenas prácticas**]]
- [[#📌 **9. Conectar VPN de Hack The Box**]]
- [[#⚙️ **10. Gestión de scripts y PATH**]]
- [[#📌 **11. Conclusiones clave**]]

---

## 🎯 **1. Introducción a FTP y su superficie de ataque**

**FTP (File Transfer Protocol)** opera típicamente en el **puerto 21/TCP** y se utiliza para:

- Subir archivos
    
- Descargar archivos
    
- Compartir recursos
    

📌 **Problema**:

- FTP **NO cifra credenciales**
    
- Configuraciones débiles son extremadamente comunes
    
- Muy frecuente en máquinas legacy y CTFs
    

---

## ⚠️ **2. Anonymous FTP login allowed**

Si durante la enumeración vemos:

```text
Anonymous FTP login allowed
```

🔥 **Esto es una vulnerabilidad directa**.

Significa que:

- Podemos autenticarnos **sin contraseña**
    
- El servidor acepta el usuario `anonymous`
    
- El password puede ser **vacío o cualquier string**
    

---

### Acceso básico

```bash
ftp <IP>
```

Cuando pida credenciales:

```text
Name: anonymous
Password: (ENTER)
```

📌 A veces acepta:

```
anonymous
anonymous@
```

---

## 🔍 **3. Enumeración de versión FTP**

Conocer la versión es clave para buscar exploits.

### Nmap

```bash
nmap -p21 -sV <IP>
```

Ejemplo de salida:

```
21/tcp open  ftp     vsftpd 2.3.4
```

📌 Algunas versiones históricamente vulnerables:

- vsftpd 2.3.4 (backdoor)
    
- ProFTPD versiones antiguas
    
- Pure-FTPd mal configurado
    

---

## 🛠 **4. Acceso y enumeración manual por FTP**

Una vez dentro:

```ftp
ls        # listar archivos
pwd       # directorio actual
cd dir    # cambiar directorio
get file  # descargar archivo
put file  # subir archivo (si permitido)
```

🔥 **Claves ofensivas**:

- Buscar `.txt`, `.log`, `.bak`
    
- Buscar credenciales
    
- Ver si permite **upload** (shells, payloads)
    

---

## 🚀 **5. Ataques de fuerza bruta con Hydra**

Si **anonymous NO está permitido**, se prueba fuerza bruta.

### Hydra FTP

```bash
hydra -l user -P rockyou.txt ftp://<IP>
```

Para múltiples usuarios:

```bash
hydra -L users.txt -P rockyou.txt ftp://<IP>
```

📌 FTP suele:

- No limitar intentos
    
- No tener fail2ban
    
- Ser ideal para brute force
    

---

## 🧠 **6. Credenciales filtradas y OSINT (DeHashed)**

Antes de brute force masivo, **piensa**.

🔗 **[https://dehashed.com](https://dehashed.com/)**

Permite:

- Buscar emails
    
- Buscar usernames
    
- Buscar passwords filtrados
    
- Relacionar servicios
    

📌 Flujo inteligente:

1. Encuentras username
    
2. Buscas en DeHashed
    
3. Pruebas credenciales reutilizadas
    
4. Acceso directo
    

---

## 🧪 **7. Escenario típico en CTF / OSCP**

1. 🔍 Nmap → puerto 21 abierto
    
2. ⚠️ Anonymous login permitido
    
3. 📂 Descargas archivos
    
4. 🔐 Encuentras credenciales
    
5. 🔄 Reutilizas credenciales en SSH / Web
    
6. 🐚 Shell obtenida
    

📌 FTP casi nunca es el final, **es el puente**.

---

## 🧱 **8. Mitigaciones y buenas prácticas**

- ❌ Deshabilitar `anonymous`
    
- ❌ No usar FTP en texto plano
    
- ✅ Migrar a SFTP
    
- ✅ Limitar IPs
    
- ✅ Permisos mínimos
    
- ✅ Logging y monitoreo
    

---

## 📌 **9. Conectar VPN de Hack The Box**

Desde el directorio donde está el `.ovpn`:

```bash
sudo openvpn ./lab_danielotero.ovpn
```

📌 Verifica conexión:

```bash
ip a
```

Debes ver `tun0`.

---

## ⚙️ **10. Gestión de scripts y PATH**

Para tener tus scripts accesibles desde cualquier lugar.

### Agregar carpeta al PATH

```bash
echo 'export PATH="$PATH:$HOME/scripts/ftp"' >> ~/.bashrc
```

📌 Puedes cambiar `ftp` por:

- `ssh`
    
- `web`
    
- `xxe`
    
- `enum`
    

Luego:

```bash
source ~/.bashrc
```

Ahora puedes ejecutar scripts **sin ruta absoluta**.

---

## 📌 **11. Conclusiones clave**

- ✔️ FTP es **antiguo y peligroso**
    
- ✔️ Anonymous login = acceso inmediato
    
- ✔️ Ideal para filtrar credenciales
    
- ✔️ Excelente punto de entrada
    
- ✔️ Siempre revisar FTP aunque parezca trivial
    

---

