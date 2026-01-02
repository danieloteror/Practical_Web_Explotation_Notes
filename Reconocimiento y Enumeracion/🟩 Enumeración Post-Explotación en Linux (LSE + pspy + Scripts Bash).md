## 📚 **Índice**

- [[#🧠 1 Importancia de la enumeración post-explotación]]
- [[#🛠️ 2 LSE Linux Smart Enumeration]]
- [[#🔍 3 pspy — Detección de procesos y tareas en tiempo real]]
- [[#💻 4 Script Bash propio para detectar tareas programadas]]
- [[#🚀 5 Flujo profesional de enumeración post-explotación]]
- [[#🧨 6 Qué buscamos realmente (vectores de escalada)]]
- [[#🛡️ 7 Medidas defensivas y buenas prácticas]]

---

# 🧠 **1. Importancia de la enumeración post-explotación**

Una vez has comprometido un sistema Linux (shell inicial, RCE, credenciales débiles, etc.),  
el siguiente objetivo es:

👉 **descubrir vías para escalar privilegios**  
👉 **persistir**  
👉 **moverte lateralmente**  
👉 **entender la estructura del sistema**

La enumeración correcta permite identificar:

- Binarios con permisos inseguros
    
- Servicios con privilegios elevados
    
- Tareas Cron vulnerables
    
- SUID misconfigurados
    
- Ficheros world-writable
    
- Contraseñas en texto plano
    
- Credenciales en backups
    
- Rutas explotables mediante PATH Hijacking
    
- Procesos que ejecutan comandos con permisos root
    

Si no enumeras bien, **te quedas atrapado como usuario básico**.

---

# 🛠️ **2. LSE — Linux Smart Enumeration**

Repositorio:  
➡ [https://github.com/diego-treitos/linux-smart-enumeration](https://github.com/diego-treitos/linux-smart-enumeration)

**LSE** es una herramienta de enumeración ligera pero extremadamente eficaz para Linux.

Funciones principales:

- 🟢 Enumera SUID/GUID peligrosos
    
- 🟢 Detecta Cron Jobs explotables
    
- 🟢 Busca binarios con permisos inseguros
    
- 🟢 Identifica configuraciones sudoers débiles
    
- 🟢 Muestra servicios, procesos y sockets
    
- 🟢 Revisa capacidades del kernel
    
- 🟢 Identifica posibles vectores de escalada
    

### 📌 Modo de uso:

Descargar:

```bash
wget https://raw.githubusercontent.com/diego-treitos/linux-smart-enumeration/master/lse.sh
ls 
```

Ejecutar:

```bash
./lse.sh
```

Modo más silencioso:

```bash
./lse.sh -l 1
```

Modo completo (ideal para pentesting):

```bash
./lse.sh -l 2
```

Información que destaca:

- Permisos raros en `/etc/sudoers`
    
- Archivos world-writable
    
- Tareas cron interesantes
    
- Binarios escalables (tar, find, perl, python, awk…)
    
- Snapshots, backups, claves SSH
    

Es la herramienta perfecta para enumerar rápido.

---

# 🔍 **3. pspy — Detección de procesos en tiempo real**

Repositorio:  
➡ [https://github.com/DominicBreuker/pspy](https://github.com/DominicBreuker/pspy)

**pspy** permite ver qué comandos se ejecutan en un sistema, incluso si no tienes root.

Es excelente para detectar:

- Cron Jobs
    
- Scripts de mantenimiento
    
- Servicios systemd que ejecutan comandos
    
- Tareas que corren con root
    
- Procesos que abren archivos sensibles
    
- Scripts con permisos de escritura que puedes modificar
    

### 📌 Descargar:

```bash
wget https://github.com/DominicBreuker/pspy/releases/download/v1.2.1/pspy64
chmod +x pspy64
```

### 📌 Ejecutar:

```bash
./pspy64
```

### ¿Qué buscamos?

- Procesos root que ejecutan scripts en `/tmp`
    
- Servicios que llaman binarios modificables
    
- Cron jobs que ejecutan comandos cada minuto
    
- Scripts sin rutas absolutas (ideal para PATH Hijacking)
    
- Comandos peligrosos como:
    

```
tar -cf backup.tar /var/www
sh /usr/local/bin/backup.sh
python3 /scripts/monitor.py
```

Cada uno puede ser un **vector de escalada a root**.

---

# 💻 **4. Script Bash propio para detectar tareas programadas**

Además de pspy, podemos escribir un script rápido para ver tareas cada cierto tiempo:

```bash
#!/bin/bash

while true; do
    ps -eo user,command >> procesos.log
    sleep 1
done
```

Es simple, pero útil en:

- Ambientes restringidos
    
- Sistemas donde pspy no ejecuta
    
- Máquinas donde solo puedes usar BusyBox
    

Una variante para imprimir solo procesos nuevos:

```bash
#!/bin/bash

OLD=""
while true; do
    NEW=$(ps -eo user,command)
    diff <(echo "$OLD") <(echo "$NEW") | grep ">"
    OLD="$NEW"
    sleep 1
done
```

Esto te muestra:

👉 “Qué se ejecutó que antes NO existía”  
Un oro para pentesters.

---

# 🚀 **5. Flujo profesional de enumeración post-explotación**

El orden correcto:

### 1️⃣ Ver quién eres

```
id
whoami
sudo -l
```

### 2️⃣ Información del sistema

```
uname -a
cat /etc/os-release
```

### 3️⃣ Usuarios y grupos

```
cat /etc/passwd
groups
```

### 4️⃣ Archivos interesantes

```
ls -la /home
find / -perm -4000 2>/dev/null
```

### 5️⃣ Tareas Cron / servicios

```
crontab -l
ls -la /etc/cron*
systemctl list-timers
```

### 6️⃣ Usar LSE para enumeración avanzada

### 7️⃣ Usar pspy para analizar procesos _dinámicos_

### 8️⃣ Buscar vectores de escalada conocidos:

- Sudo misconfigurado
    
- SUID binarios
    
- Capabilities
    
- Cron jobs vulnerables
    
- PATH Hijacking
    
- Writeable scripts ejecutados por root
    

---

# 🧨 **6. Qué buscamos realmente (vectores de escalada)**

El objetivo no es “ver procesos” → sino **encontrar un camino a root**.

Los vectores reales:

### ✔ Cron Jobs mal configurados

Script ejecutado por root que puedes modificar.

### ✔ SUID peligrosos

Ejemplo:

```
/usr/bin/find
/usr/bin/python
/usr/bin/vim
```

Con ellos puedes escalar así:

```bash
find . -exec /bin/sh \; -quit
```

### ✔ Archivos con permisos 777 en rutas críticas

### ✔ Configuraciones sudoers inseguras

### ✔ PATH Hijacking

Si root ejecuta:

```
backup
```

Y no usa rutas absolutas → tú puedes inyectar un binario falso.

### ✔ Servicios systemd vulnerables

---

# 🛡️ **7. Medidas defensivas y buenas prácticas**

Un administrador debería:

- Remover permisos SUID innecesarios
    
- Asegurar scripts en `/usr/local/bin`
    
- Usar paths absolutos en Cron jobs
    
- Bloquear acceso a herramientas peligrosas
    
- Monitorizar procesos sospechosos
    
- Usar auditd
    

Pero como atacante, estos fallos son **tu escalera hacia root**.

---

# 🎯 **Conclusión PRO**

Una vez tienes acceso a un sistema, tu poder depende del nivel de enumeración que hagas.

- **LSE** → información estructurada
    
- **pspy** → información dinámica
    
- **Scripts Bash** → apoyo en entornos restringidos
    

Juntos te permiten:

✔ detectar vectores  
✔ escalar privilegios  
✔ comprometer completamente el sistema

---
