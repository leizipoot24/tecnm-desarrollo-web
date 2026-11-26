# Bitácora de Requerimientos y Auditoría IA — Sesión 17
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** Control de Acceso Basado en Roles (RBAC)  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Leizi Astril Poot Cabrera]  
**Matrícula:** [23531123]  
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Verificación manual de permisos de usuario (Admin/User)  
*Describe qué dificultad encontraste al procesar la petición HTTP o validar campos a mano:*
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | Utilicé Python, FastAPI y Google Colab para desarrollar una API con autenticación JWT y control de acceso. |
| **Rol (R)** | La IA debe actuar como desarrollador backend especializado en FastAPI, JWT y seguridad mediante roles. |
| **Especificación (E)** |Crear dos usuarios, admin y user; generar tokens JWT con sus roles y proteger las rutas para distinguir entre 401 y 403. |
| **Output (O)** | Un archivo app.py funcional con las rutas /api/token, /api/admin/panel y /api/perfil. |

### 2.2 Prompt Estructurado
```text
[Ayudame a realizar esta actividad paso a paso guiandome y explicando a detalle]
```

---

## 3. Código Devuelto por el Modelo
```python
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
## VERIFICACIÓN DEL RESULTADO · 6 casos · app.py

==============================================================================

✅ 01. El administrador obtiene su token

✅ 02. El usuario normal obtiene su token

✅ 03. El admin entra al panel

✅ 04. El usuario normal recibe 403

✅ 05. Sin token recibe 401

✅ 06. El perfil funciona para el usuario normal

------------------------------------------------------------------------------

RESULTADO: 6/6 casos correctos. ¡Resultado logrado!

---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Pruebas de acceso denegado por rol insuficiente (HTTP 403)

* [x] Respuestas semánticas correctas (`200 OK`, `201 Created`, `400 Bad Request`, `422 Unprocessable`).
* [x] No existen respuestas HTTP `500 Internal Server Error` no capturadas.
* [x] Encabezados `Content-Type: application/json` correctamente configurados.

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
La IA no había diferenciado correctamente los errores HTTP 401 Unauthorized y 403 Forbidden. Lo corregí implementando una validación para que el sistema responda con 401 cuando no se proporciona un token válido y con 403 cuando un usuario autenticado intenta acceder a una ruta que requiere permisos de administrador. Con esta corrección, las seis pruebas se ejecutaron correctamente.

---
**Firma digital:** [Leizipoot24]
