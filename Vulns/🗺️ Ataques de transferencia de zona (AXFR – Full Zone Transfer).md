## 📚 **Índice**

- [[#🔥 1. Concepto real de los ataques DNS Zone Transfer (AXFR/IXFR)]]
- [[#🌍 2. Cómo funciona DNS por dentro (visión rápida y práctica)]]
- [[#⚠️ 3. Por qué existe la vulnerabilidad (root cause técnica)]]
- [[#💥 4. Cómo explota un atacante AXFR/IXFR]]
- [[#🧪 5. Comandos ofensivos reales con dig (AXFR e IXFR)]]
- [[#🔍 6. Cómo detectar si un dominio es vulnerable]]
- [[#🚀 7. Casos de aplicación REAL en pentesting]]
- [[#🧵 8. Cómo usar la información filtrada en ataques posteriores]]
- [[#🛡️ 9. Mitigaciones correctas y configuración segura]]
- [[#🐳 10. Laboratorio vulnerable en Docker para practicar]]
- [[#📎 11. Recursos adicionales]]

---

# 🔥 **1. Concepto real de los ataques DNS Zone Transfer (AXFR/IXFR)**

Una _Zone Transfer_ es un mecanismo de DNS para replicar contenido entre servidores autoritativos:

- **AXFR** → Transferencia completa de zona
    
- **IXFR** → Transferencia incremental de zona
    

El problema:

> Si un servidor DNS está misconfigurado, un atacante puede solicitar una transferencia completa de zona y recibir **toda la base de datos DNS del dominio**, incluyendo subdominios internos, IP privadas, MX, TXT, registros SPF, etc.

Este ataque permite **reconstruir la topología completa de un dominio**.

---

# 🌍 **2. Cómo funciona DNS por dentro (visión rápida y práctica)**

Un servidor DNS autoritativo tiene un **zone file** que contiene:

- Subdominios
    
- IPs asociadas
    
- MX (correo)
    
- NS
    
- TXT
    
- SPF
    
- Registros internos que _no deberían ser públicos_
    

Ejemplo interno:

```
intranet.company.com    A   10.10.50.21
dev-api.company.com     A   172.31.23.50
mail.company.com        MX  mail.company.com
vpn.company.com         A   20.55.10.110
```

Una transferencia de zona expone **todo esto**.

---

# ⚠️ **3. Por qué existe la vulnerabilidad (root cause técnica)**

La causa raíz siempre es:

### ❌ 1. Servidor DNS configurado para aceptar AXFR desde cualquier origen

### ❌ 2. Falta de control de acceso en servidores secundarios

### ❌ 3. DNS legacy que nunca fue endurecido

### ❌ 4. Administradores que desconocen que AXFR debe limitarse a IPs autorizadas

Muchos servidores siguen exponiendo AXFR porque:

- usan configuraciones heredadas
    
- fueron mal clonados
    
- están en modo debug
    
- están mal configurados en entornos cloud
    

---

# 💥 **4. Cómo explota un atacante AXFR/IXFR**

El atacante envía una consulta especial:

```
AXFR → Pido TODA la zona
IXFR → Pido cambios incrementales (menos común pero útil)
```

Si el DNS responde:

- Obtienes **toda la estructura interna del dominio**
    
- Sabes qué IPs existen
    
- Descubres **hostnames internos**
    
- Descubres entornos ocultos como:
    
    - staging
        
    - dev
        
    - preprod
        
    - VPN
        
    - bases de datos expuestas
        
    - paneles admin
        
- Puedes mapear infraestructura completa en segundos
    

Este ataque es literalmente **un treasure map** para pentesters.

---

# 🧪 **5. Comandos ofensivos reales con dig (AXFR e IXFR)**

## 🔥 **AXFR — Transferencia completa de zona**

```
dig @<DNS-server> <domain-name> AXFR
```

Ejemplo real:

```
dig @ns1.target.com target.com AXFR
```

Si devuelve registros → dominio vulnerable.

---

## 🔁 **IXFR — Transferencia incremental**

Para probar IXFR:

```
dig @<DNS-server> <domain-name> IXFR
```

Ejemplo:

```
dig @ns2.target.com target.com IXFR=1
```

IXFR a veces funciona cuando AXFR está bloqueado.

---

## 🎯 **Uso rápido**

```
dig axfr @<ip> <domain>
```

Ejemplo práctico:

```
dig axfr @192.168.1.53 example.com
```

---

# 🔍 **6. Cómo detectar si un dominio es vulnerable**

Método ofensivo estándar:

### ✔ Paso 1: Enumerar nameservers (NS)

```
dig <domain> NS
```

### ✔ Paso 2: Probar AXFR contra cada servidor

```
dig @ns1.domain.com domain.com AXFR
dig @ns2.domain.com domain.com AXFR
dig @ns3.domain.com domain.com AXFR
```

### ✔ Paso 3: Probar IXFR si AXFR falla

```
dig @ns1.domain.com domain.com IXFR=1
```

### ✔ Indicadores de vulnerabilidad:

- El servidor responde con **todos los registros DNS**
    
- El servidor responde con **datos internos (10.x.x.x / 172.16.x.x)**
    
- El servidor revela **subdominios no públicos**
    
- Se obtiene la confirmación de SOA + NS + A, MX, TXT, etc.
    

Si AXFR funciona → **la organización está expuesta gravemente**.

---

# 🚀 **7. Casos de aplicación REAL en pentesting**

## 🧨 **1. Descubrimiento de subdominios internos**

Subdominios tipo:

```
dev.company.com
staging.company.com
gitlab.company.com
intranet.company.com
vpn.company.com
db01.company.com
```

Con eso haces:

- SSRF
    
- RCE
    
- Escalada interna
    
- Pivoting
    
- Bruteforce de paneles internos
    

---

## 🧨 **2. Mapeo de infraestructura**

La transferencia de zona te puede revelar:

- IP privadas internas (AWS, Azure, LAN)
    
- Rangos de red
    
- Hosts activos
    
- Jerarquía interna de servicios
    

---

## 🧨 **3. Identificación de servidores de correo (MX)**

Puedes apuntar ataques:

- Phishing preciso
    
- Spoofing
    
- Ataques a servidores obsoletos
    

---

## 🧨 **4. Descubrir TXT sensibles**

Muchos admins guardan cosas como:

```
"internalId=43234"
"dev-key=SECRET"
"spf: include:internal.mail.server"
```

---

## 🧨 **5. Pivot hacia otros ataques**

La información recolectada sirve como base para:

- Subdomain Takeover
    
- Ataques contra CPanel/ISPConfig
    
- Descubrimiento de S3 buckets
    
- Envenenamiento DNS
    
- Ataques sobre CDN
    
- Ataques a VPN empresarial
    

---

# 🧵 **8. Cómo usar la información filtrada en ataques posteriores**

### ✔ Identificar hosts para escaneo específico (masscan/nmap)

### ✔ Encontrar paneles internos expuestos en internet

### ✔ Localizar servicios mal configurados

### ✔ Descubrir versiones antiguas de servidores DNS

### ✔ Identificar dominios abandonados o takeoverables

---

# 🛡️ **9. Mitigaciones correctas y configuración segura**

### 🔐 Limitar AXFR a IPs autorizadas de servidores secundarios

Ejemplo BIND:

```
allow-transfer { 192.168.1.10; };
```

### 🚫 Bloquear AXFR público

```
allow-transfer { none; };
```

### 🔏 DNSSEC + TSIG para autenticación

### 🧱 Firewall para restringir TCP/53

### 📡 Monitoreo de logs DNS

Si ves consultas AXFR → es un indicador de ataque.

---

# 🐳 **10. Laboratorio vulnerable en Docker para practicar**

Repositorio (Vulhub):

```
https://github.com/vulhub/vulhub/tree/master/dns/dns-zone-transfer
```

Permite practicar:

- AXFR
    
- IXFR
    
- NS enumeration
    
- Configuración vulnerable
    
- Cómo se ve una transferencia completa
    

---

# 📎 **11. Recursos adicionales**

- RFC 5936 — DNS Zone Transfer
    
- BIND 9 Documentation
    
- dig manual
    
- OWASP DNS Security Testing Guide
    
- HackTricks — DNS Enumeration
    

---

# 🎯 **Mini resumen PRO 

### ✔ Paso 1: Tienes un dominio

Ej: `empresa.com`

### ✔ Paso 2: Obtienes NS
```
`dig empresa.com NS`
```
Resultado:

- `ns1.empresa.com`
    
- `ns2.empresa.com`
    
- `ns3.proveedorDNS.net`
    

### ✔ Paso 3: Intentas AXFR
```
dig @ns1.empresa.com empresa.com AXFR 
dig @ns2.empresa.com empresa.com AXFR 
dig @ns3.proveedorDNS.net empresa.com AXFR
```
### ✔ Paso 4: Si responde → vulnerable

Ejemplo típico de éxito:

`; Transfer started. 
dev.empresa.com          A   10.10.10.30
mail.empresa.com         MX  10 mail.empresa.com 
vpn.empresa.com          A   44.22.11.90
; Transfer completed.`

Cuando ves esto:  
👉 **LA EMPRESA ESTÁ DESNUDA.**