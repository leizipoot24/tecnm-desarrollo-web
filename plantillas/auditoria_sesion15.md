# Bitácora de Requerimientos y Auditoría IA — Sesión 15
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** Autenticación Stateless con JSON Web Tokens (JWT)  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Leizi Astril Poot Cabrera]  
**Matrícula:** [23531123]  
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Codificación Base64Url y firma HMAC-SHA256  
*Describe qué dificultad encontraste al procesar la petición HTTP o validar campos a mano:*
Mi mayor dificultad fue comprender cómo se genera y valida un token JWT, porque tuve que entender cómo funciona la firma y cómo comprobar que el token no haya sido modificado. También se me complicó manejar correctamente las respuestas cuando los datos eran incorrectos.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | Python, FastAPI y PyJWT trabajando en Google Colab. |
| **Rol (R)** | 	Asistente técnico especializado en desarrollo de APIs y seguridad web. |
| **Especificación (E)** | Crear un endpoint para generar tokens JWT y otro para validar el acceso al perfil del usuario. |
| **Output (O)** | Código funcional en Python, utilizando modelos Pydantic, validaciones y respuestas HTTP correctas. |

### 2.2 Prompt Estructurado
```text
[Ayudame a realizar esta actividad guiando paso a paso y explicando lo que estoy haciendo]
```

---

## 3. Código Devuelto por el Modelo
```python
# 
%%writefile app.py

import os
from datetime import datetime, timedelta, timezone

import jwt
from jwt.exceptions import InvalidTokenError

from fastapi import FastAPI, HTTPException, Depends
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from pydantic import BaseModel


app = FastAPI(title="Sistema de Autenticación JWT")

JWT_SECRET = os.getenv("JWT_SECRET_KEY")

if not JWT_SECRET:
    raise RuntimeError("Falta configurar JWT_SECRET_KEY")

ALGORITHM = "HS256"
TIEMPO_EXPIRACION = 30

USUARIOS = {
    "ana": "secreta"
}

security = HTTPBearer(auto_error=False)


class LoginEntrada(BaseModel):
    usuario: str
    password: str


@app.post("/api/token")
def generar_token(datos: LoginEntrada):

    if USUARIOS.get(datos.usuario) != datos.password:
        raise HTTPException(
            status_code=401,
            detail="Usuario o contraseña incorrectos"
        )

    ahora = datetime.now(timezone.utc)

    payload = {
        "sub": datos.usuario,
        "exp": ahora + timedelta(minutes=TIEMPO_EXPIRACION)
    }

    token = jwt.encode(
        payload,
        JWT_SECRET,
        algorithm=ALGORITHM
    )

    return {
        "access_token": token,
        "token_type": "bearer"
    }


def obtener_usuario(
    credentials: HTTPAuthorizationCredentials = Depends(security)
):

    if credentials is None:
        raise HTTPException(
            status_code=401,
            detail="Token no proporcionado",
            headers={"WWW-Authenticate": "Bearer"}
        )

    try:
        payload = jwt.decode(
            credentials.credentials,
            JWT_SECRET,
            algorithms=[ALGORITHM]
        )

        usuario = payload.get("sub")

        if not usuario:
            raise InvalidTokenError("Usuario no encontrado")

        return usuario

    except InvalidTokenError:
        raise HTTPException(
            status_code=401,
            detail="Token inválido o vencido",
            headers={"WWW-Authenticate": "Bearer"}
        )


@app.get("/api/perfil")
def obtener_perfil(usuario: str = Depends(obtener_usuario)):

    return {
        "usuario": usuario
    }

```

import os
import secrets

os.environ["JWT_SECRET_KEY"] = secrets.token_urlsafe(32)

print("Clave JWT configurada correctamente")

---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Pruebas con tokens alterados o firma inválida

* [x] Respuestas semánticas correctas (`200 OK`, `201 Created`, `400 Bad Request`, `422 Unprocessable`).
* [x] No existen respuestas HTTP `500 Internal Server Error` no capturadas.
* [x] Encabezados `Content-Type: application/json` correctamente configurados.

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
Durante la revisión comprendí que era importante controlar los errores cuando se enviaba un token alterado, vencido o cuando no se proporcionaba ningún token. Se debía responder con HTTP 401 y no permitir el acceso al perfil. También utilicé una variable de entorno para proteger la clave secreta y evitar escribirla directamente en el código.

---
**Firma digital:** [leizipoot24]
