# Bitácora de Requerimientos y Auditoría IA — Sesión 14
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** Hashing Seguro de Contraseñas (Bcrypt / Argon2)  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Leizi Astril Poot Cabrera]  
**Matrícula:** [23531123]  
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Cifrado manual con Salting y Factor de Costo  
*Describe qué dificultad encontraste al procesar la petición HTTP o validar campos a mano:*
La dificultad que encontré fue comprender cómo proteger las contraseñas sin guardarlas directamente. También me resultó complicado validar los datos recibidos y comprobar que la contraseña ingresada fuera correcta sin tener que recuperar la original. Después comprendí que bcrypt facilita este proceso porque genera un hash con un salt y permite verificarlo mediante una función especial.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | Python 3 y FastAPI, utilizando Google Colab y la biblioteca bcrypt para proteger las contraseñas. |
| **Rol (R)** | Desarrollador backend especializado en APIs REST y seguridad web, encargado de implementar un sistema de registro e inicio de sesión seguro. |
| **Especificación (E)** | Crear las rutas POST /api/registro y POST /api/login. Utilizar bcrypt para generar y verificar hashes, validar los datos recibidos, evitar usuarios duplicados y no exponer contraseñas ni hashes. |
| **Output (O)** | Archivo app.py funcional, modelos Pydantic, respuestas JSON y códigos HTTP 201, 200, 401 y 409. |

### 2.2 Prompt Estructurado
```text
[Ayudame a realizar esta actividad guiandome y explicando paso a paso esta actividad ]
```

---

## 3. Código Devuelto por el Modelo
```python
%%writefile app.py

from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, Field, ConfigDict
import bcrypt

app = FastAPI(title="Sistema de Registro y Login Seguro")

usuarios_db = {}

class UsuarioEntrada(BaseModel):
    usuario: str = Field(min_length=1)
    password: str = Field(min_length=1)

    model_config = ConfigDict(str_strip_whitespace=True)

class RegistroSalida(BaseModel):
    usuario: str

class LoginSalida(BaseModel):
    usuario: str
    autenticado: bool

@app.post("/api/registro", response_model=RegistroSalida, status_code=201)
def registrar_usuario(datos: UsuarioEntrada):

    if datos.usuario in usuarios_db:
        raise HTTPException(
            status_code=409,
            detail="El usuario ya existe"
        )

    password_hash = bcrypt.hashpw(
        datos.password.encode("utf-8"),
        bcrypt.gensalt()
    )

    usuarios_db[datos.usuario] = password_hash.decode("utf-8")

    return RegistroSalida(usuario=datos.usuario)


@app.post("/api/login", response_model=LoginSalida, status_code=200)
def iniciar_sesion(datos: UsuarioEntrada):

    password_hash = usuarios_db.get(datos.usuario)

    if password_hash is None:
        raise HTTPException(
            status_code=401,
            detail="Credenciales incorrectas"
        )

    password_correcta = bcrypt.checkpw(
        datos.password.encode("utf-8"),
        password_hash.encode("utf-8")
    )

    if not password_correcta:
        raise HTTPException(
            status_code=401,
            detail="Credenciales incorrectas"
        )

    return LoginSalida(
        usuario=datos.usuario,
        autenticado=True
    )

import os
from fastapi.testclient import TestClient
from app import app

print("ARCHIVO APP.PY:", os.path.exists("app.py"))

cliente = TestClient(app)

registro = cliente.post(
    "/api/registro",
    json={"usuario": "ana", "password": "secreta"}
)

login_correcto = cliente.post(
    "/api/login",
    json={"usuario": "ana", "password": "secreta"}
)

login_incorrecto = cliente.post(
    "/api/login",
    json={"usuario": "ana", "password": "incorrecta"}
)

usuario_repetido = cliente.post(
    "/api/registro",
    json={"usuario": "ana", "password": "otra"}
)

print("REGISTRO:", registro.status_code, registro.json())
print("LOGIN CORRECTO:", login_correcto.status_code, login_correcto.json())
print("LOGIN INCORRECTO:", login_incorrecto.status_code)
print("USUARIO REPETIDO:", usuario_repetido.status_code)

assert registro.status_code == 201
assert login_correcto.status_code == 200
assert login_incorrecto.status_code == 401
assert usuario_repetido.status_code == 409

assert "password" not in registro.json()
assert "hash" not in registro.json()
assert "password" not in login_correcto.json()
assert "hash" not in login_correcto.json()

print("\nTODAS LAS PRUEBAS CORRECTAS")

```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Pruebas contra ataques de Rainbow Tables y Timing

* [x] Respuestas semánticas correctas (`200 OK`, `201 Created`, `400 Bad Request`, `422 Unprocessable`).
* [x] No existen respuestas HTTP `500 Internal Server Error` no capturadas.
* [x] Encabezados `Content-Type: application/json` correctamente configurados.

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
Al revisar el código comprendí que era importante evitar que un usuario pudiera registrarse varias veces. Para solucionarlo agregué el código HTTP 409 Conflict cuando el usuario ya existe. También utilicé modelos de respuesta para evitar que se mostraran datos privados como la contraseña o el hash. Durante las primeras pruebas tuve un problema porque el verificador no encontraba el archivo app.py, por lo que tuve que volver a crear el archivo y ejecutar nuevamente las pruebas. 

---
**Firma digital:** [leizipoot24]
