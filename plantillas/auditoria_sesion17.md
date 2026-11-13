# Bitácora de Requerimientos y Auditoría IA — Sesión 17
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** Control de Acceso Basado en Roles (RBAC)  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Leizi astril Poot Cabrera]  
**Matrícula:** [23531123]  
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Verificación manual de permisos de usuario (Admin/User)  
*Describe qué dificultad encontraste al procesar la petición HTTP o validar campos a mano:*
La dificultad que encontré fue diferenciar cuándo un usuario no está autenticado y cuándo sí inició sesión, pero no tiene permiso para entrar a una ruta. También tuve que comprobar que cada usuario tuviera el rol correcto y que el token JWT incluyera esa información.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** |FastAPI con Python 3.10 o superior, utilizando Google Colab para ejecutar la práctica. |
| **Rol (R)** | Arquitecto de software especializado en APIs REST y seguridad web.|
| **Especificación (E)** | Implementar autenticación JWT con dos roles, admin y user, incluyendo protección de rutas mediante los permisos correspondientes. |
| **Output (O)** | Código modular en Python, validación de tokens, control de roles y respuestas HTTP 200, 401 y 403. |

### 2.2 Prompt Estructurado
```text
[ayudame a realizar esta actividad guiando y veriificando paso a paso lo qeu estoy haciendo, explicame a detalle]
```

---

## 3. Código Devuelto por el Modelo
```python
#
%%writefile app.py

from datetime import datetime, timedelta, timezone
from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer
from pydantic import BaseModel
from jose import JWTError, jwt

app = FastAPI(title="Sistema de Control de Acceso RBAC")

CLAVE_SECRETA = "clave_secreta_sesion17"
ALGORITMO = "HS256"
TIEMPO_EXPIRACION = 30

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/api/token")


class LoginEntrada(BaseModel):
    usuario: str
    password: str


USUARIOS = {
    "admin": {
        "usuario": "admin",
        "password": "admin123",
        "rol": "admin"
    },
    "ana": {
        "usuario": "ana",
        "password": "secreta",
        "rol": "user"
    }
}


def crear_token(usuario):
    ahora = datetime.now(timezone.utc)

    datos = {
        "sub": usuario["usuario"],
        "rol": usuario["rol"],
        "exp": ahora + timedelta(minutes=TIEMPO_EXPIRACION)
    }

    return jwt.encode(datos, CLAVE_SECRETA, algorithm=ALGORITMO)


@app.post("/api/token")
def obtener_token(datos: LoginEntrada):
    usuario = USUARIOS.get(datos.usuario)

    if not usuario or usuario["password"] != datos.password:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Credenciales incorrectas",
            headers={"WWW-Authenticate": "Bearer"}
        )

    return {
        "access_token": crear_token(usuario),
        "token_type": "bearer"
    }


def obtener_usuario_actual(token: str = Depends(oauth2_scheme)):
    try:
        datos = jwt.decode(
            token,
            CLAVE_SECRETA,
            algorithms=[ALGORITMO]
        )

        nombre = datos.get("sub")
        rol = datos.get("rol")

        usuario = USUARIOS.get(nombre)

        if not usuario or usuario["rol"] != rol:
            raise HTTPException(
                status_code=status.HTTP_401_UNAUTHORIZED,
                detail="Token inválido",
                headers={"WWW-Authenticate": "Bearer"}
            )

        return {
            "usuario": nombre,
            "rol": rol
        }

    except JWTError:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Token inválido o expirado",
            headers={"WWW-Authenticate": "Bearer"}
        )


def requiere_admin(usuario=Depends(obtener_usuario_actual)):
    if usuario["rol"] != "admin":
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="No tienes permisos de administrador"
        )

    return usuario


@app.get("/api/admin/panel")
def panel_admin(usuario=Depends(requiere_admin)):
    return {"panel": "admin"}


@app.get("/api/perfil")
def perfil(usuario=Depends(obtener_usuario_actual)):
    return usuario

```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Pruebas de acceso denegado por rol insuficiente (HTTP 403)

* [x] Respuestas semánticas correctas (`200 OK`, `201 Created`, `400 Bad Request`, `422 Unprocessable`).
* [x] No existen respuestas HTTP `500 Internal Server Error` no capturadas.
* [x] Encabezados `Content-Type: application/json` correctamente configurados.

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
Durante la revisión comprendí que era importante separar los errores 401 y 403. Un usuario sin token debe recibir 401, mientras que un usuario autenticado que no tiene el rol de administrador debe recibir 403. Para corregirlo, separé la validación del token de la comprobación del rol, utilizando dos funciones diferentes. Así evité confundir la autenticación con la autorización.

---
**Firma digital:** [leizipoot24]
