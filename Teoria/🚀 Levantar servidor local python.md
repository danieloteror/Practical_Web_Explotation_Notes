# 1) Servir una carpeta (Python 3, recomendado)

Abre la terminal en la carpeta que quieres exponer y ejecuta:

```bash
# desde dentro del directorio que quieres servir
python3 -m http.server 8000
```

Esto arranca un servidor en el puerto `8000`. Abres en el navegador `http://localhost:8000/` y verás los archivos.

Opciones útiles:

```bash
# especificar puerto y bind a localhost (más seguro)
python3 -m http.server 8000 --bind 127.0.0.1

# servir otra carpeta sin hacer cd (Python 3.7+)
python3 -m http.server 8000 --directory /ruta/a/tu/carpeta --bind 127.0.0.1
```

---

# 2) Servir una carpeta (Python 2 — obsoleto)

Si solo tienes Python 2:

```bash
python -m SimpleHTTPServer 8000
```

(usa Python 3 siempre que puedas).

---

# 3) Servir **solo un archivo** (ejemplo sencillo)

Si quieres exponer _únicamente_ un archivo concreto (por ejemplo `secreto.txt`), usa este script mínimo (Python 3):

```python
# serve_one_file.py
from http.server import HTTPServer, BaseHTTPRequestHandler
from pathlib import Path
import urllib

FILE = Path("/ruta/a/secreto.txt")  # <- ruta al archivo que quieres servir

class OneFileHandler(BaseHTTPRequestHandler):
    def do_GET(self):
        # servir solo la raíz o el nombre del archivo
        req_path = urllib.parse.unquote(self.path)
        if req_path in ("/", f"/{FILE.name}"):
            if FILE.exists():
                self.send_response(200)
                self.send_header("Content-Type", "application/octet-stream")
                self.send_header("Content-Length", str(FILE.stat().st_size))
                self.end_headers()
                with FILE.open("rb") as f:
                    self.wfile.write(f.read())
            else:
                self.send_error(404, "File not found")
        else:
            self.send_error(403, "Forbidden")

if __name__ == "__main__":
    server = HTTPServer(("127.0.0.1", 8000), OneFileHandler)
    print("Serving", FILE, "on http://127.0.0.1:8000/")
    server.serve_forever()
```

Guardar como `serve_one_file.py` y ejecutar:

```bash
python3 serve_one_file.py
```

Esto expone solo `/` o `/secreto.txt` en `http://127.0.0.1:8000/`.

---

# 4) Notas de seguridad y buenas prácticas

- **No** uses estos servidores para producción. Son útiles para pruebas y transferencias locales.
    
- Si vas a exponer a la red, usa `--bind 127.0.0.1` para limitar al localhost o configura firewall.
    
- Revisa permisos de archivos: el servidor entrega lo que esté en la carpeta.
    
- Si necesitas HTTPS o autenticación, usa soluciones más completas (nginx, Caddy, o frameworks con TLS).
    

---

Si quieres, puedo:

- darte el comando exacto para Windows PowerShell,
    
- preparar un `systemd` service simple para ejecutar el script (si lo vas a usar en una máquina Linux),
    
- o crear un ejemplo que sirva múltiples archivos con autenticación básica.
    

¿Quieres alguno de esos?